In this final part of the lab, I will:
- Use Kali Linux to perform a brute force attack
- Use Splunk to detect the attack and view the telemetry
- Set up and install Atomic Red Team
- Run Atomic Red Team and detect telemetry using that.

First, I will log into Kali Linux. In a previous section of the lab, note that I set the IP address to 192.168.10.250. I can check it with `ip addr` in the terminal to verify.

![](/Screenshots/Pasted%20image%2020260223142711.png)

Next, I will update with `sudo apt-get update && sudo apt get upgrade -y`

![](/Screenshots/Pasted%20image%2020260223142831.png)

Now I will set up a new directory called ad-project using `mkdir ad-project`.

![](/Screenshots/Pasted%20image%2020260223143012.png)

Next, install crowbar with `sudo apt-get install -y crowbar`. This will be the tool I use to perform brute force attacks against my windows machine. 

![](/Screenshots/Pasted%20image%2020260223143817.png)

Next, we will go to the wordlists folder to be able to use a popular wordlist file, rockyou.txt

![](/Screenshots/Pasted%20image%2020260223144009.png)

Next, unzip it with gunzip with `sudo gunzip rockyou.txt.gz`

![](/Screenshots/Pasted%20image%2020260223144042.png)

After that, copy it to the folder we made previously with `cp rockyou.txt ~/ad-project`

![](/Screenshots/Pasted%20image%2020260223144400.png)

From there, go to the same folder using `cd ~/ad-project/`

![](/Screenshots/Pasted%20image%2020260223144512.png)

This file has many many passwords, and this is just a demo, so I will pretend that, as a 'hacker', I am targeting the specific 'super secure' password associated with Philip J Fry's account. I still want it try try many passwords, so I will create a file called password.txt from the top 20 passwords in the rockyou.txt file using `head -n rockyou.txt > password.txt`. Then, using `nano password.txt`, I will put the machine's password at the bottom and save it using ctrl + x and y to confirm.

![](/Screenshots/Pasted%20image%2020260223145312.png)

![](/Screenshots/Pasted%20image%2020260223145300.png)

I can view the passwords using `cat password.txt`

![](/Screenshots/Pasted%20image%2020260223145704.png)

Back on the Windows 10 machine we are targeting, I am going to enable remote desktop. Type in "PC" at the bottom, then on the "This PC" option, click properties.

![](/Screenshots/Pasted%20image%2020260223145834.png)

Scroll to the bottom and go to "Advanced system settings"

![](/Screenshots/Pasted%20image%2020260223145900.png)

Sign in with the administrator account.

![](/Screenshots/Pasted%20image%2020260223145931.png)

Go to the "Remote" tab at the top, click "Allow remote connections to this computer, then click "Select Users".

![](/Screenshots/Pasted%20image%2020260223150012.png)

Click "Add"

![](/Screenshots/Pasted%20image%2020260223150025.png)

Type in 'pfry' then click "Check Names". It should autopopulate. Press "OK", "OK", then "Apply".

![](/Screenshots/Pasted%20image%2020260223150053.png)

![](/Screenshots/Pasted%20image%2020260223150123.png)

Back on the Kali machine, we're going to use our new tool (crowbar) to try to do a brute force attack. 

![](/Screenshots/Pasted%20image%2020260223150625.png)

Huh... that's a little weird. The script is not working even though it's got the correct password. Let me check below with nmap.

![](/Screenshots/Pasted%20image%2020260224131316.png)

I used `nmap 192.168.10.102/32` to target only that IP address and the port's definitely open. I wonder what could be causing this?

![](/Screenshots/Pasted%20image%2020260224131415.png)

Searching on Google, it looks like NLA (Network Level Authentication) could be the culprit. It says that some crowbar versions do not handle this very well. I can run an nmap script to check. If the script below shows CredSSP or NLA, this is probably the problem.

`nmap -p 3389 --script rdp-enum-encryption 192.168.10.102`

![](/Screenshots/Pasted%20image%2020260224131803.png)

Ah, so it shows both. Let me change some settings on the Windows 10 machine so I can continue.

Back in Windows under System Settings > Advanced System Configuration > Remote, I will uncheck the box below that says "Allow connections only from computers running Remote Desktop with Network Level Authentication (recommended)" and click "Apply". 

![](/Screenshots/Pasted%20image%2020260224132047.png)

Alright, let's try Crowbar again. I'll test with just the password itself and then try the wordlist if it works.

![](/Screenshots/Pasted%20image%2020260225142600.png)

![](/Screenshots/Pasted%20image%2020260225142617.png)

Okayyy... This is not going to be as straightforward as I thought. I think I need to go off-script here and use a different tool.

Doing some research, I've found a tool named Hydra that may be able to assist. I will use the following command to see if it works:

`hydra -l pfry -P password.txt rdp://192.168.10.102 -e nsr`

![](/Screenshots/Pasted%20image%2020260225142846.png)

Hey! It works! That is great! Now we've 'hacked' this machine and used a wordlist to brute force the password. That should generate some telemetry on Splunk. 

Back on the Windows 10 Machine, I am going to Splunk at 192.168.10.10:8000 and logging in. Once logged in, I am going to go to "Search & Reporting"

![](/Screenshots/Pasted%20image%2020260225144330.png)

I am going to search `index=endpoint pfry` and set the time to the last 60 minutes.
![](/Screenshots/Pasted%20image%2020260225144424.png)

Once searched, I will open the field names and look for "EventCode"

![](/Screenshots/Pasted%20image%2020260225144531.png)

Looks like there were 128 events for 4625. But what does it mean? We can search this up on Google to find out.

![](/Screenshots/Pasted%20image%2020260225144604.png)

Here's what it says on Microsoft Learn:

![](/Screenshots/Pasted%20image%2020260225144703.png)

This actually tracks well, because behind the scenes, I did end up running Crowbar several times before I gave up and tried Hydra instead. 

Next, let's look up event code 4624, which has a count of 28 times:

![](/Screenshots/Pasted%20image%2020260225145016.png)

Looks like this one means it was logged into successfully. Finally, we can go to the bottom of the fields on the side and find 'Workstation_Name'.

![](/Screenshots/Pasted%20image%2020260225145110.png)

Now remember, Splunk doesn't have a universal forwarder associated with the Kali machine.. which means it's detecting a login from the Kali machine on the Windows 10 target machine.

![](/Screenshots/Pasted%20image%2020260225145231.png)

And if we click Kali, we see the events below, stating that the login WAS in the target-PC under account name pfry. 

![](/Screenshots/Pasted%20image%2020260225145334.png)

Next, I'm going to install ART(Atomic Red Team) as the final portion of this project. 

First, I need to go back to the Target PC and run a PowerShell command. Run PowerShell as administrator, then put in the admin credentials. After that, run this command:

`Set-ExecutionPolicy Bypass CurrentUser`

Next, I will make an exclusion for the C drive in the security settings, so that Windows doesn't remove ART files from the system.  Click the little up arrow in the bottom right, then go to the security tab. 

![](/Screenshots/Pasted%20image%2020260225145712.png)

Go to "Virus & Threat Protection"
![](/Screenshots/Pasted%20image%2020260225145811.png)

Go to "Manage Settings"

![](/Screenshots/Pasted%20image%2020260225145830.png)

Go to "Add or Remove Exclusions" and fill in the Administrator credentials when prompted.

![](/Screenshots/Pasted%20image%2020260225145901.png)

Click "Add an exclusion" and "Folder"

![](/Screenshots/Pasted%20image%2020260225145938.png)

Go to "This PC", scroll down, and click the C drive.

![](/Screenshots/Pasted%20image%2020260225150000.png)

There! Now the exclusion has been made. 

Back in Powershell, I ran the following commands:

`IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invokeatomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);`

`Install-AtomicRedTeam -getAtomics`

And then confirmed with 'Y' when prompted. 

Now, going back to the C drive, I can go to AtomicRedTeam > Atomics and see the different 'atomics' (MITRE ATT&CK framework techniques). 

![](/Screenshots/Pasted%20image%2020260225151351.png)

What these do is allow us to test different types of attacks that we may be vulnerable to and see if we have any blind spots. If you go to attack.mitre.org, there are several techniques there that attackers might use. 

So let's try to see how it works. Going to attack.mitre.org, I will hover over "Command and Scripting Interpreter" to see what the technique number is.

![](/Screenshots/Pasted%20image%2020260225152118.png)

It's T1059. Hovering over 'Powershell' gives you the first technique, T1059.001.

Looks like we have that technique below, so we can test it.
![](/Screenshots/Pasted%20image%2020260225151801.png)

I will run the command below:

![](/Screenshots/Pasted%20image%2020260225152339.png)

It will start running many tests. When I ran this, my Windows Defender tripped!

![](/Screenshots/Pasted%20image%2020260225152628.png)

Back on Splunk, we can see that it has detected the test:

![](/Screenshots/Pasted%20image%2020260225153222.png)

Atomic Red Team is helpful in generating results like this so that you can see what it would look like in Splunk, or if Splunk does not detect it at all, so that you can adjust your alerts so that it will detect things like this, covering the gaps that your Splunk would not see. 

And that's the entire project done! Yay!

In the future, I will use the Active Directory Domain Controller to practice more on Active Directory itself, hack our Windows 10 Machine and Server more with Kali, and much more. Stay tuned for more projects!