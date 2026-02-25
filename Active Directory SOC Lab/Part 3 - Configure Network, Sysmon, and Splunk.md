Alright, so now we need to create the network and give each device their IP addresses. Consider the diagram we made in the first exercise:
![](Pasted%20image%2020260220130313.png)
So to recap, we need:
Network: 192.168.10.0/24
Splunk(Ubuntu) Server: 192.168.10.10
Active Directory (Windows Server 2022): 192.168.10.7
Attacker (Kali): 192.168.10.250
Windows 10: DHCP on the 192.168.10.0/24 range

To begin, I need to make a linux bridge in Proxmox.  I'm going to number the steps in Proxmox as they may be a little hard to follow. 
1. Click your host
2. Click Network
3. Click "Create" and choose "Linux Bridge"
![](Pasted%20image%2020260221124724.png)

4. Input the name 'vmbr1' or another name you can remember and the IP/CIDR 192.168.10.1/24 
5. Click 'Create' and 'Apply Configuration' at the top once you've created the bridge.

![](Pasted%20image%2020260221125229.png)

Next, we need to enable network address translation (NAT) on the Proxmox host if we want the devices to have internet access. To do so, I need to go to the configuration file on the host itself and add vmbr1 to post-up and post-down.
6. Go to the host
7. Click "Shell"
8. Type `nano /etc/network/interfaces` and press enter

![](Pasted%20image%2020260221125553.png)

9. Add the following to the vmbr1 entry:
post-up   echo 1 > /proc/sys/net/ipv4/ip_forward  
post-up   iptables -t nat -A POSTROUTING -s '192.168.10.0/24' -o vmbr0 -j MASQUERADE  
post-down iptables -t nat -D POSTROUTING -s '192.168.10.0/24' -o vmbr0 -j MASQUERADE

![](Pasted%20image%2020260221125938.png)

10. Bring the interface up using `ifup vmbr1`. Use the command `ifreload -a` to apply the configuration if necessary.

![](Pasted%20image%2020260221130321.png)

11. Next, I'll install and run a dhcp server in Proxmox for the Windows machine. `apt install isc-dhcp-server` will get us a server, but it will need to be configured.

![](Pasted%20image%2020260221130633.png)

12. After this, set the INTERFACESv4 to vmbr1:  
`sed -i 's/INTERFACESv4=""/INTERFACESv4="vmbr1"/' /etc/default/isc-dhcp-server`

![](Pasted%20image%2020260221143014.png)

13. Next, add this configuration to declare a subnet for the network in /etc/dhcp/dhcpd.conf:
`nano /etc/dhcp/dhcpd.conf`

```
subnet 192.168.10.0 netmask 255.255.255.0 {
	  range 192.168.10.100 192.168.10.200;
      option routers 192.168.10.1;
      option domain-name-servers 1.1.1.1;
  }
```

![](Pasted%20image%2020260221143210.png)

From here, we need to configure each VM to use vmbr1; it is the same on all machines, so here is an example using the Windows10 machine:

14. Click "pve"
15. Click the machine you want to switch the network device over to (make sure to do this for all 4 VMs, one at a time.)
16. Click "Network Device"

![](Pasted%20image%2020260221143348.png)

17. Select 'vmbr1'
18. Click 'OK'

![](Pasted%20image%2020260221143437.png)

Again, make sure to do this for the Kali, Splunk-Ubuntu, and Windows Server 2022 machines as well. The steps are exactly the same for each with the exception of selecting the different machines themselves. 

Back on Windows 10, this should pop up; accept it and it should give you an IP address.

![](Pasted%20image%2020260221131712.png)

Next, we can confirm in the command prompt that it gives us an IP within the range with `ipconfig` . Since the range selected was 192.168.10.100 - 192.168.10.200, it gave us 192.168.10.102. 

![](Pasted%20image%2020260221131926.png)

Good! Windows 10 is configured with DHCP. Now, any time we connect a machine to the bridge, it will get a DHCP address if we do not assign it a static address. Now we can configure Windows server 2022, Ubuntu, and Kali Linux.

Over on Windows Server, go to the Control Panel > Network and Sharing Center > Change adapter settings.

![](Pasted%20image%2020260221132451.png)

Right click the "Ethernet" option and click properties, choose "Internet Protocol Version 4", and click "Configure." Click "Use the following IP address" to set a static IP.

![](Pasted%20image%2020260221132618.png)

Fill in the following information:

IP Address: 192.168.10.7
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
Preferred DNS Server: 1.1.1.1
Alternate DNS Server: 8.8.8.8

And press 'OK'.

![](Pasted%20image%2020260221133143.png)

That's it for the Windows Server network configuration. We can check in the command prompt with `ipconfig` if needed. Next, I'll do Ubuntu, then Kali.

First, we can check the current configuration in Ubuntu.

`ls /etc/netplan`

For me, it's stored under '/etc/netplan/50-cloud-init.yaml'. So I will check the configuration with `cat /etc/netplan/50-cloud-init.yaml'`

![](Pasted%20image%2020260221133909.png)

Next, I will edit the configuration with a static IP. First, I will nano the file using `sudo nano /etc/netplan/50-cloud-init.yaml`. Then, I will apply the following configuration:

```
network:  
   ethernets:  
     ens18:  
       addresses: [192.168.10.10/24]  
       routes:  
         - to: default  
           via: 192.168.10.1  
       nameservers:  
         addresses: [1.1.1.1, 8.8.8.8]
    version: 2        
```
![](Pasted%20image%2020260221140523.png)

Then use `sudo netplan apply` to apply the configuration. Use `ip addr` to check and confirm you've got it set correctly:

![](Pasted%20image%2020260221140642.png)

After that, we can go to Kali. This one is fairly easy as Kali uses NetworkManager by default. The simplest way is via nmcli in the terminal. First, I use `nmcli con show` to find my interface name:

![](Pasted%20image%2020260221141544.png)

Looks like it's named Wired connection 1. Now, I'll apply this configuration using the name I just found:

```
 nmcli con mod "Wired connection 1" \
      ipv4.addresses 192.168.10.250/24 \
      ipv4.gateway 192.168.10.1 \
      ipv4.dns "1.1.1.1,8.8.8.8" \
      ipv4.method manual

```

![](Pasted%20image%2020260221141553.png)

Then, use `nmcli con up "Wired connection 1"` to complete the connection: 

![](Pasted%20image%2020260221141704.png)

We can check our connection with `ip addr`

![](Pasted%20image%2020260221141735.png)

Presto! Looks like we've got all of our IP addresses correctly configured.

Now, let's install Splunk. To do so, we need to make an account on splunk.com, sign in, and go to the "Trials and Downloads" section. 

![](Pasted%20image%2020260221144636.png)

Next, choose "Start download" under Splunk Enterprise. 

![](Pasted%20image%2020260221144724.png)

Make sure you choose "Linux", and choose the ".deb" package. You can either Download it and use some tricks to get it downloaded to your proxmox machine (for example, you could use rsync) or you can do it the manual way with a wget link. I simply wrote the wget link into Ubuntu manually. This is because Proxmox doesn't allow for direct copy-paste unless you install a few plugins to make that work, and since I won't need to directly copy-paste a lot for this project, I decided not to include that this time.

![](Pasted%20image%2020260221144929.png)

Next, use `sudo dpkg -i` and point it at the splunk install file in Ubuntu.

![](Pasted%20image%2020260221145102.png)

Next, type in `cd /opt/splunk && ls -la` to view the folder with splunk in it.

![](Pasted%20image%2020260221145311.png)

Next, change to the user "splunk" with `sudo -u splunk bash`, then `cd /opt/splunk/bin` to get into the binaries folder, and finally `./splunk start` to run the installer. Press 'Y' when at the end to agree to the license agreement.

![](Pasted%20image%2020260222153447.png)
![](Pasted%20image%2020260221145710.png)

Next, put in a username and a super secure password.

After that, it will install. Type 'exit' to exit the user 'splunk' when finished. Next, `cd bin`, and finally, `sudo ./splunk enable boot-start -user splunk` to enable splunk every time the VM boots.

![](Pasted%20image%2020260221150416.png)

Back on the Windows10 machine, we're going to rename it. 

![](Pasted%20image%2020260221150548.png)

I will click here and name it target-PC. It will have you restart after this.

![](Pasted%20image%2020260221150625.png)

Next, we can install Splunk Universal Forwarder on this machine. Head to https://www.splunk.com on this virtual machine and log in there, then search for 'Splunk Universal Forwarder' and download it. Run the installation program and check the box to accept the license agreement. 

![](Pasted%20image%2020260221151356.png)

![](Pasted%20image%2020260221151439.png)

Skip this step since we don't have a deployment server:

![](Pasted%20image%2020260221151522.png)

On this step, we put the IP of the Ubuntu machine and the port 9997.

![](Pasted%20image%2020260221151606.png)

Next, while this is installing, we can download sysmon from sysinternals with the following link:

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Next, we'll download the sysmon Olaf configuration. Make sure it saves to your Downloads folder.

https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml

Once done, extract the sysmon .zip file to the folder it was downloaded in:
![](Pasted%20image%2020260221152146.png)

Next, we'll open Windows Powershell as Administrator and install it using that Olaf configuration. First, navigate to the Sysmon directory using `cd C:\Users\Windows10\Downloads\Sysmon` . Then, run the following command. 

`\.Sysmon.exe -i ..\sysmonconfig.xml`

![](Pasted%20image%2020260221152331.png)

Agree to the software license terms and it will install.
![](Pasted%20image%2020260221152602.png)

Next, we need to configure what gets sent to Splunk. We can go to C:\Program Files\SplunkUniversalForwarder\etc\system\default\ and copy inputs.conf. DO NOT EDIT IT.

![](Pasted%20image%2020260221152831.png)

Go back one folder, then go to the 'local' folder, and paste the inputs.conf file there.

![](Pasted%20image%2020260221152917.png)

Next, we need to edit that file. We can open it with notepad, delete what's there, and put this in, instead: 

```
[WinEventLog://Application]

index = endpoint

disabled = false

[WinEventLog://Security]

index = endpoint

disabled = false

[WinEventLog://System]

index = endpoint

disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]

index = endpoint

disabled = false

renderXml = true

source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

After that, save it. Then go to the start bar and type in 'Services' and run it as administrator. 

![](Pasted%20image%2020260221153656.png)

Find 'SplunkForwarder', right click it, and go to 'Properties' then click 'Log On'.

![](Pasted%20image%2020260221153801.png)

Change it to 'Local System Account', click "OK", then restart the service.

![](Pasted%20image%2020260221153854.png)

Next, open a web browser and go to 192.168.10.10:8000 and log into Splunk with the account we made on Ubuntu earlier. Go to 'Settings' and click 'Indexes'.

![](Pasted%20image%2020260221154143.png)

Create a 'New Index'. Name it 'endpoint' to match the configuration file from earlier.

![](Pasted%20image%2020260221154234.png)

Next, go to 'Settings' and click 'Forwarding and receiving'

![](Pasted%20image%2020260221154342.png)

Click "Configure receiving", "New receiving port" and enter "9997"

![](Pasted%20image%2020260221154459.png)
![](Pasted%20image%2020260221154526.png)

Once done, head over to the "Search" page and type in `index="endpoint"`. If all is configured correctly, you should see events!

![](Pasted%20image%2020260221154736.png)

Next, I moved over to Windows Server 2022. I renamed the Windows Server 2022 ADDC01 the same way I renamed the Windows 10 PC. Once it rebooted, I did the exact same process as with the Windows 10 PC to install Splunk Universal Forwarder and Sysmon with the Olaf configuration. If done correctly, it should show under 'hosts' when you search `index="endpoint"`

![](Pasted%20image%2020260221160504.png)

And now, this part of the project is done. You are now seeing events from both Windows devices on the Splunk server.

