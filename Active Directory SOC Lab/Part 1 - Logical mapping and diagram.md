This will be a lab to practice both Active directory and some key cybersecurity concepts. I am using MyDFIR's Active Directory Project on YouTube to go through the concepts; however, he is doing it in VirtualBox, whereas I thought it would be a nice change to do it on a Proxmox server in order to not completely copy his videos and have some challenges I'll need to work out on my own. 

This project's Active Directory installation will also be used in later projects in order to learn and show my skills working with Active Directory Domain Services. I will also install a Windows 11 VM in the future to be able to show the differences between 10 and 11 logs.

My main resources will be MyDFIR's Active Directory Project videos Google, and Proxmox's documentation on their website. 

The first step is to use Draw.io to make our diagram:
![](/Screenshots/Pasted%20image%2020260220130313.png)

So as we can see, our information is as follows:
Domain: AdamsDomain
Network: 192.168.10.0/24
VM IPs:
Splunk: 192.168.10.10
Active Directory/Windows Server: 192.168.10.7
Kali Linux: 192.168.10.250
Windows 10: DHCP 

There are some other requirements:
- All of the machines will need to connect to a virtual bridge that acts as a switch and router/gateway
- The virtual bridge will need to connect all of these machines to the internet
- The Windows Server 2022 and Windows 10 VMs need Sysmon, the Sysmon Olaf Configuration, and Splunk Universal Forwarder installed
- Windows Server 2022 and Windows 10 will need to forward logs over to the Splunk server
- Our Splunk Server needs to actually receive those logs from both servers

Alright! Let's go ahead and start on the next part of the project in Part 2.
