In order to be able to fulfill the objectives of this lab, I will need to set up 4 virtual machines on my Proxmox server. 

Here's the plan for part 2:
1. Install Windows Server
2. Install Ubuntu server
3. Install Kali Linux
4. Install Windows 10 

First, I installed Proxmox 9.1.4. onto a physical server for virtualization purposes. Since this lab mainly focuses on Active Directory and Splunk, I will not go over the installation process for Proxmox in this writeup. After that, I went to the following websites to download the respective ISO files for the virtual machines I will be running:

Microsoft Windows 10
https://www.microsoft.com/en-us/software-download/windows10ISO

Windows Server 2022
https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022

Ubuntu Server 
https://ubuntu.com/download/server

Kali Linux
https://www.kali.org/get-kali/#kali-installer-images

I downloaded the 64-bit version for each of them and continued the process. First, I installed Windows 10. I'm going to number the steps in Proxmox as they may be a little hard to follow.

1. Click Create VM
2.  Name the VM. For simplicity, I named mine "Windows10"
3. Pick a VM ID. I chose 110.

Then, go to the next page. 
![](/Screenshots/Pasted%20image%2020260220155550.png)

4. Choose the storage the ISO was saved into. I uploaded mine to local storage.
5. Choose the windows 10 ISO
6. Choose Microsoft Windows
7. Choose Windows 10

Then, go to the next page.

![](/Screenshots/Pasted%20image%2020260220155941.png)

8. Choose 'q35' under "Machine"
9. Choose OVMF (UEFI) under "BIOS"
10. Check the "Add EFI Disk" option and choose a disk for EFI storage
11. Choose "Raw disk image (raw)" (this is just to see the difference between qcow2 and raw)
12. Check the "Add TPM" box (not necessary, but I will need to do this later for Windows 11 machines anyways, so good to get into the habit)
13. Choose the TPM Storage device

Then, go to the next page.

![](/Screenshots/Pasted%20image%2020260220160423.png)

14. Choose storage for your disk
15. Change the disk size to 50GB 
16. Choose "Raw disk image" under "Format"

Then, go to the next page.

![](/Screenshots/Pasted%20image%2020260220160732.png)

17. Set the cpu Cores to 1, then go to the next page.

![](/Screenshots/Pasted%20image%2020260220161344.png)

18. Set the RAM to 4096 MiB. Click "Next" and leave the "Network" selections as default for now, as we will change those later.

![](/Screenshots/Pasted%20image%2020260220161414.png)

19. Click "Start after created", then click "Finish"

![](/Screenshots/Pasted%20image%2020260220161836.png)

You may get this screen; press any button to continue, then choose the "UEFI QEMU DVD-ROM" selection and press any key when prompted.

![](/Screenshots/Pasted%20image%2020260220162128.png)
![](/Screenshots/Pasted%20image%2020260220162139.png)

Click Next. When prompted, select "Install Now", then, "I don't have a product key"

![](/Screenshots/Pasted%20image%2020260220162216.png)
![](/Screenshots/Pasted%20image%2020260220162437.png)
![](/Screenshots/Pasted%20image%2020260220162317.png)

Select "Windows 10 Pro"

![](/Screenshots/Pasted%20image%2020260220162347.png)

Accept the license terms.

![](/Screenshots/Pasted%20image%2020260220162414.png)

Choose "Custom: Windows only (advanced)"

![](/Screenshots/Pasted%20image%2020260220162531.png)

Select Drive 0.

![](/Screenshots/Pasted%20image%2020260220162548.png)

From here, it will install Windows 10. When it is done, we will finish up the last few steps.

![](/Screenshots/Pasted%20image%2020260220162616.png)

Choose "United States"

![](/Screenshots/Pasted%20image%2020260220163013.png)

Choose the keyboard layout.

![](/Screenshots/Pasted%20image%2020260220163027.png)

Skip the next step.

![](/Screenshots/Pasted%20image%2020260220163041.png)

Select "Set up for Personal use" and click next.

![](/Screenshots/Pasted%20image%2020260220163145.png)

Choose "Offline account"

![](/Screenshots/Pasted%20image%2020260220163223.png)

Choose "Limited Experience"

![](/Screenshots/Pasted%20image%2020260220163251.png)

Name the OS. I named it "Windows10"

![](/Screenshots/Pasted%20image%2020260220163309.png)

Make a super secure password and confirm it on the next page.


![](/Screenshots/Pasted%20image%2020260220163359.png)

Choose some security questions and enter the answers

![](/Screenshots/Pasted%20image%2020260220163515.png)


Choose "Not now" on the next page

![](/Screenshots/Pasted%20image%2020260220163541.png)

I personally prefer to turn off all of the things on the next page.

![](/Screenshots/Pasted%20image%2020260220163616.png)

Skip the next page.

![](/Screenshots/Pasted%20image%2020260220163637.png)

Click "Not now" on the next page.

![](/Screenshots/Pasted%20image%2020260220163701.png)

After that, you will need to wait several minutes for the Windows installation to finalize. When you see install log in, you are done with Windows 10 for now.

![](/Screenshots/Pasted%20image%2020260220163902.png)

Next, we install Kali Linux. We could either use an already premade custom image, or install it from scratch. I'll install it from scratch. 

Click "Create VM" again, then follow these steps:
1. Choose a VM ID. I chose "111"
2. Name your VM. I named mine "Kali"

Then, go to the next screen. 
![](Screenshots/Pasted%20image%2020260225163358.png)

3. Choose the storage where the ISO is kept.
4. Choose the Kali Linux ISO.

Then, go to the next screen.

![](/Screenshots/Pasted%20image%2020260220164212.png)

Keep the defaults here and go to the next screen.

![](/Screenshots/Pasted%20image%2020260220164255.png)

Keep the defaults here and go to the next screen again.

![](/Screenshots/Pasted%20image%2020260220164346.png)

5. Give the machine 2 CPU cores.  

![](/Screenshots/Pasted%20image%2020260220164442.png)

6. Give the machine 2048 MiB (2 gigabytes) of Memory.
 
![](/Screenshots/Pasted%20image%2020260220164522.png)

Keep the network information the same for now, then click "Next" to get to the confirm page. 
6. Check the box that says "Start after created" and click Finish.

![](/Screenshots/Pasted%20image%2020260220164628.png)

Choose "Graphical Install"

![](/Screenshots/Pasted%20image%2020260220164720.png)

Choose "English"

![](/Screenshots/Pasted%20image%2020260220165045.png)

Choose "United States"

![](/Screenshots/Pasted%20image%2020260220165107.png)

Choose "American English"

![](/Screenshots/Pasted%20image%2020260220165126.png)

Then wait for it to configure the installation. This may take some time. After that, give the system a hostname. I chose the default, "kali"

![](/Screenshots/Pasted%20image%2020260220165213.png)

Leave the next option blank and hit 'Continue'.

![](/Screenshots/Pasted%20image%2020260220165325.png)

Put in your name.

![](/Screenshots/Pasted%20image%2020260220165347.png)

Create an account name.

![](/Screenshots/Pasted%20image%2020260220165359.png)

Make a super strong password.

![](/Screenshots/Pasted%20image%2020260220165418.png)

Choose your timezone.

![](/Screenshots/Pasted%20image%2020260220165431.png)

Leave the next few options as default unless you really want to use LVM or encryption. Since these are not within the scope of this lab, I left mine as default. 

![](/Screenshots/Pasted%20image%2020260220165454.png)
![](/Screenshots/Pasted%20image%2020260220165505.png)
![](/Screenshots/Pasted%20image%2020260220165518.png)
![](/Screenshots/Pasted%20image%2020260220165527.png)

Select "Yes" on this page and hit "Continue" to confirm the changes.

![](/Screenshots/Pasted%20image%2020260220165537.png)

Leave the defaults on this page and hit 'Continue'. It will install kali and the tools selected, which will take some time. 

![](/Screenshots/Pasted%20image%2020260220165807.png)

After that, it will ask you to install Grub Bootloader, and tell you to confirm the disk you wanted to install it to.

![](/Screenshots/Pasted%20image%2020260220170923.png)
![](/Screenshots/Pasted%20image%2020260220170935.png)

When you have reached the page below, you are good to go!

![](/Screenshots/Pasted%20image%2020260220171018.png)

You should be able to log into Kali Linux here.

![](/Screenshots/Pasted%20image%2020260220171107.png)

Next, we will create the Windows Server 2022 virtual Machine.
1. Choose a VM ID. I chose "115"
2. Name your server.

![](/Screenshots/Pasted%20image%2020260220170315.png)

3. Choose the storage the ISO is located in
4. Choose the ISO image
5. Choose "Microsoft Windows"
6. Choose version "11"

![](/Screenshots/Pasted%20image%2020260220170617.png)

7. Your previous installation of Windows should have most of these at a good default. Choose "q35" if not selected
8. Choose "OVMF (UEFI)"
9. Check the box that says "Add EFI Disk"
10. Choose the storage device for your EFI disk
11. Check the "Add TPM" box
12. Choose the storage for your TPM device

![](/Screenshots/Pasted%20image%2020260220170738.png)

13. Increase the disk size to 50 gigabytes
 
![](/Screenshots/Pasted%20image%2020260220170807.png)

14. Give the machine 2 cores.

![](/Screenshots/Pasted%20image%2020260220170822.png)

15. Give the machine 4 gigabytes of RAM.

![](/Screenshots/Pasted%20image%2020260220170841.png)

16. Keep the network settings as default, click next, then check the "Start after created" box and click finish.

![](/Screenshots/Pasted%20image%2020260220170900.png)

Again, choose the "UEFI QEMU DVD-ROM" selection then press a key when prompted.

![](/Screenshots/Pasted%20image%2020260220171752.png)

Click "Next".

![](/Screenshots/Pasted%20image%2020260220171838.png)

Click "Install now"

![](/Screenshots/Pasted%20image%2020260220171846.png)

Choose "Windows Server 2022 Standard Evaluation (Desktop Experience)" so we have a GUI, otherwise it would be a command-prompt based experience.

![](/Screenshots/Pasted%20image%2020260220171917.png)

Accept the Microsoft Software License Terms.

![](/Screenshots/Pasted%20image%2020260220171931.png)

Choose "Custom"

![](/Screenshots/Pasted%20image%2020260220171944.png)

Choose the disk to store things on. After this, it will start installing (this may take some time).

![](/Screenshots/Pasted%20image%2020260220171954.png)

Make a super strong password.

![](/Screenshots/Pasted%20image%2020260220172240.png)

You should be greeted with this screen. Notice how much cleaner and smoother this process is than Windows 10.

![](/Screenshots/Pasted%20image%2020260220172317.png)

Once you've logged in, you will get to this screen:

![](/Screenshots/Pasted%20image%2020260220172534.png)

This part is done for now. Now we can move on to making a Splunk server with Ubuntu.

Again, choose "Create VM" at the top of the screen. Then. do the following. 
 
 1. Choose a VM ID. I chose '112'
 2. Choose a name. I chose 'Splunk-Ubuntu'
  
![](/Screenshots/Pasted%20image%2020260220172828.png)

3. Choose the storage that your ISO is stored in
4. Choose the Ubuntu ISO. On the next page, leave the defaults and click 'Next'.

![](/Screenshots/Pasted%20image%2020260220172858.png)
![](/Screenshots/Pasted%20image%2020260220172920.png)

5. Increase the disk size to 100GB. 
 
![](/Screenshots/Pasted%20image%2020260220173006.png)

6. Give the machine 2 CPU cores.

![](/Screenshots/Pasted%20image%2020260220173057.png)

7. Give the machine 8 gigabytes of RAM.

![](/Screenshots/Pasted%20image%2020260220174622.png)

8. Check the "Start after created" box and then click Finish.

![](/Screenshots/Pasted%20image%2020260220174647.png)

Choose the first option.

![](/Screenshots/Pasted%20image%2020260220173228.png)

Choose "English"

![](/Screenshots/Pasted%20image%2020260220173302.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173319.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173331.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173344.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173404.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173446.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173502.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173530.png)

Press "Continue"

![](/Screenshots/Pasted%20image%2020260220173542.png)

Put in your name, your server's name, and a username and password. 

![](/Screenshots/Pasted%20image%2020260220173624.png)

Choose "Continue"

![](/Screenshots/Pasted%20image%2020260220173636.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173707.png)

Choose "Done"

![](/Screenshots/Pasted%20image%2020260220173732.png)

Wait for it to install, then choose "Reboot Now"

![](/Screenshots/Pasted%20image%2020260220173756.png)

You may get the error below. You can either press a key to skip it, or if it doesn't allow you to, remove the installation media.

![](/Screenshots/Pasted%20image%2020260220173820.png)

Here I am doing so in proxmox. I selected the ISO, then "Remove" at the top.

![](/Screenshots/Pasted%20image%2020260220173838.png)

From here, log in, then run the following command:
`sudo apt-get update && sudo apt-get upgrade -y`

![](/Screenshots/Pasted%20image%2020260220174028.png)

That's it! All of the VMs are installed and ready to use. In the next section, I will configure the network, sysmon, and Splunk Universal Forwarder on both the Windows 10 VM and the Windows Server 2022 VM.

