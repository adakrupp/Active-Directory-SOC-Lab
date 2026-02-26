In this part of the project, I will:
- Configure Active Directory with ADDS and promote it to a domain controller
- Create a new domain
- Create some organizational units (OUs) 
- Create some users.  
- Configure the Windows 10 machine to be able to log into my domain
- Log into a user account with the Windows 10 machine. 

Before, we set the IP to 192.168.10.7 -- we can check it in the command prompt with `ipconfig`.

![](/Screenshots/Pasted%20image%2020260222160451.png)

Next, I'm going to ping the Splunk server to make sure there's connectivity. If there is, that also means that there should be connectivity to the Windows 10 VM (pinging Windows 10 directly would not work since ICMP is blocked by default on Windows)

![](/Screenshots/Pasted%20image%2020260222160609.png)

Perfect! Time to set up ADDS with the Server Manager.

![](/Screenshots/Pasted%20image%2020260222160651.png)

Server Manager should pop up immediately upon login. From here, select "Manage", then "Add Roles and Features". Click Next.

![](/Screenshots/Pasted%20image%2020260222160733.png)

Make sure it shows "Role-based or feature-based installation". 

![](/Screenshots/Pasted%20image%2020260222160809.png)

If there were multiple servers, we'd have more than one to choose from here; however, we only have the one. so we can click next. 

![](/Screenshots/Pasted%20image%2020260222160838.png)

Select "Active Directory Domain Services", then "Add Features", then "Next" and keep clicking "Next" until we get to the "Install" option.

![](/Screenshots/Pasted%20image%2020260222160908.png)![](/Screenshots/Pasted%20image%2020260222160917.png)![](/Screenshots/Pasted%20image%2020260222160928.png)
![](/Screenshots/Pasted%20image%2020260222161056.png)
![](/Screenshots/Pasted%20image%2020260222161115.png)

When it says "Installation Succeeded" it is ready to go. Next, we will need to promote the server to a domain controller. We can do so with the flag icon in the upper right corner of Server Manager.

![](/Screenshots/Pasted%20image%2020260222161231.png)
![](/Screenshots/Pasted%20image%2020260222161257.png)

Click "Add a new forest", then name it. I named mine "AdamsDomain.local". Then click "Next".

![](/Screenshots/Pasted%20image%2020260222161406.png)

Leave the defaults here and put in a super secure password. Then click next until you get to the "Install" option.

![](/Screenshots/Pasted%20image%2020260222161502.png)
![](/Screenshots/Pasted%20image%2020260222161637.png)
![](/Screenshots/Pasted%20image%2020260222161654.png)![](/Screenshots/Pasted%20image%2020260222161704.png)![](/Screenshots/Pasted%20image%2020260222161727.png)

It will require you to restart the server after this. Go ahead and restart when prompted.

![](/Screenshots/Pasted%20image%2020260222161816.png)

Upon logging in, you will see your domain's name next to the Administrator account. 

![](/Screenshots/Pasted%20image%2020260222162134.png)

Alright! Let's log in and create some users! Log in, then click "Tools" on Server Manager, then click "Active Directory Users and Computers."

![](/Screenshots/Pasted%20image%2020260222162244.png)

Here, there are several pre-populated groups in 'Builtin', but they cannot be edited. 

![](/Screenshots/Pasted%20image%2020260222162313.png)

We can use these if we want to in order to automatically authorize users to do certain things. I will experiment with these in a future project, but for now, I just need to create some new OUs and users.

![](/Screenshots/Pasted%20image%2020260222162422.png)

In order to simulate a real-world environment, we will create several OUs, or Organizational Units. These are to simulate departments, such as "IT", "HR", "Finance," "Customer Service", "Management", etc, that you would find in an actual business. Each of these can then be given certain permissions depending on their function. For now, I will create a few OUs so that I can use them to make users for this project.

First, right click the domain, then go to new, then Organizational Unit. I will make 5 based on the above.

![](/Screenshots/Pasted%20image%2020260222163029.png)

Within each unit, I will create a user. They will be:

IT: Bender Rodriguez
HR: Amy Wong
Finance: Hermes Conrad
Management: Hubert Farnsworth
Customer Service: Philip J. Fry

First, I will right click each unit, then click New > User. In future projects, I will automate this with scripts.

![](/Screenshots/Pasted%20image%2020260222163508.png)

After that, I will fill in their first and last names, and give them a username. After that, click next.

![](/Screenshots/Pasted%20image%2020260222163633.png)

In the following page, I'm going to uncheck the box that says "User must change password at next login"; however, in the real world, we'd want to leave that checked. Also notice that we can leave the account disabled, make it so the user can't change the password, or make it so their password does not expire. Click "Next", then "Finish" on the next page.

![](/Screenshots/Pasted%20image%2020260222163746.png)
![](/Screenshots/Pasted%20image%2020260222163842.png)

Now our new user shows up in the IT department. I will do the same for each user in each department.

![](/Screenshots/Pasted%20image%2020260222163935.png)
![](/Screenshots/Pasted%20image%2020260222164153.png)![](/Screenshots/Pasted%20image%2020260222164200.png)![](/Screenshots/Pasted%20image%2020260222164212.png)
![](/Screenshots/Pasted%20image%2020260222164226.png)

Woohoo! I've made a bunch of OUs and made users within each of them. Next, I will go to my Windows 10 VM and join it to the domain, then authenticate and log in with Philip J Fry's account. 

First, I go to the Windows 10 machine and go to This PC > Properties

![](/Screenshots/Pasted%20image%2020260222164610.png)

Scroll down to "Advanced system settings".

![](/Screenshots/Pasted%20image%2020260222164645.png)

Go to "Computer Name" at the top and select "Change".

![](/Screenshots/Pasted%20image%2020260222164736.png)

Click "Domain" and type in the domain name.

![](/Screenshots/Pasted%20image%2020260222164824.png)

If you click "OK" it will give us an error. 

![](/Screenshots/Pasted%20image%2020260222164913.png)

We can fix that in the Network settings. Leave this window up and right click the network icon in the bottom, then go to "Open Network & Internet Settings"

![](/Screenshots/Pasted%20image%2020260222165037.png)

Click "Change adapter options", right click the Ethernet connection and go to "Properties",  and go to the IPV4 option, click it, and click "Properties".

![](/Screenshots/Pasted%20image%2020260222165213.png)![](/Screenshots/Pasted%20image%2020260222165235.png)
![](/Screenshots/Pasted%20image%2020260222165322.png)

Change the DNS options to "Use the following DNS server address" and type in the IP address of the Windows Server 2022 machine. Then, go back to the window you left up and click "OK" to join the domain. 

![](/Screenshots/Pasted%20image%2020260222165501.png)
![](/Screenshots/Pasted%20image%2020260222165542.png)

Type in the Administrator information on the following popup and press "OK". In a real world scenario, you'd use a custom user that is not the administrator in the proper group (Like IT) that are able to allow others to join our domain. For now, I will go ahead and use the administrator account.

![](/Screenshots/Pasted%20image%2020260222165633.png)

You will be greeted with the popup below:

![](/Screenshots/Pasted%20image%2020260222165655.png)

It will then have you restart the computer. After that, we will try to sign in with Philip J. Fry's account information.

![](/Screenshots/Pasted%20image%2020260222165920.png)
![](/Screenshots/Pasted%20image%2020260222165941.png)

Yay! We've done it!

![](/Screenshots/Pasted%20image%2020260222170007.png)

Alright, now that I've made several OUs and users, joined a machine to a domain, then logged into a user on that machine, I now have a fully functional Active Directory domain that will be used in future Active Directory-specific projects. The next part of the project, Part 5, is the final part that I will do for this particular project. In that part, I will be brute-forcing the Windows 10 Machine with Kali Linux, checking Splunk telemetry, and installing and using Atomic Red Team.