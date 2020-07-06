# Kali Linux WSL2 Setup

Guide on how to setup the perfect pentesting environment using Kali Linux running on WSL2 for windows

## Get WSL2

More information on how to get WSL2 can be found at: https://docs.microsoft.com/en-us/windows/wsl/install-win10

### Update Windows

Check your version of windows with this command: `winver`  

Make sure you have version 2004 or later (build 19041 or higher). However, if you do not you can check for updates in the Windows Update Centre, alternatively download the Windows Upgrade Assistant from here and clicking 'Update Now': https://www.microsoft.com/en-gb/software-download/windows10

### Enable WSL

Open PowerShell as Administrator and run:  
`dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`

Restart your PC.

### Set WSL 2 as your default version

Open PowerShell and run:  
`wsl --set-default-version 2`

## Get Kali Linux

### Download Kali

Open the Microsoft Store and search for "Kali Linux", here is a link to the store page: https://www.microsoft.com/store/apps/9PKR34TNCV07

### Install Kali

Open PowerShell and run:  
`kali`  
You will then need to create a user account and password.

### Install Kali tools and software

First update the repos and existing software. From bash run:  
`sudo apt update && sudo apt upgrade -y`  
When this has finished install the default Kali Linux tools by running:  
`sudo apt install kali-linux-default`

If you would prefer to install a different set of tools you can check the list of Kali metapackages here: https://www.kali.org/news/major-metapackage-makeover/

That's it! You now have a fully functioning Kali Linux install... Well almost :/ Some features won't work out of the box due to how WSL starts the Linux system. You may run into this error at some point:  
`System has not been booted with systemd as init system (PID 1). Can't operate.`  
`Failed to connect to bus: Host is down`

This is perfectly normal, there are some ways around this but they are a little more involved. These are shown later in the guide if you want to try them at your own risk.

## Opening GUI Apps

There are times when using WSL when you may want to interact with the GUI of a particular tool or web browser etc. How to do this is not immediately apparent, but it can be done quite easily. There are a number of ways to accomplish this however, I have found the way that gives the best experience is by using an X Server. This is where your WSL installation of Kali will forward the graphical display output of programs that require one to a server running on your host PC. Follow the steps to set it up.

### Install VcXsrv Windows X Server

Go to this link:  
https://sourceforge.net/projects/vcxsrv/  
Click download. When the installer has downloaded, proceed with the installation by following the wizard.

### Run VcXsrv

From the start menu search for "XLaunch" and open the program. Another wizard will show up, follow the steps to start the server.  

- Multiple Windows, Display number: -1
- Start no client
- TICK ALL BOXES! - Make sure to disable access control
- Finish

The X Server will now be running in the background, you will see a new 'X' icon in your system tray.

### Setup display forwarding in kali

First find the local IPV4 address of your host PC. From PowerShell run:  
`ipconfig`  
Take note of your IP address.

Now from the bash terminal for your Kali Linux run:  
`export DISPLAY=<YOUR IP HERE>:0`  
Make sure to substitute in the IP you took note of.

Add this into your .bashrc file to export this variable automatically when you login:  
`echo DISPLAY=<YOUR IP HERE>:0 >> ~/.bashrc`  
Making sure again to substitute your host PC's IP address.

Your Host PC might not have a static local IPV4 address. So if this ever changes, run "ipconfig" on Windows again and edit your bashrc file accordingly. Run:  
`nano ~/.bashrc`  
Scroll to the bottom and edit your IP address.

That's it! Now we can test it. Install a web browser of your choice from the bash command line. To install Firefox run:  
`sudo apt install firefox-esr`  
When it has finished installing run the command to launch the browser. To launch Firefox run:  
`firefox`  
If all has worked correctly you should see a new window opens up in Windows. This will look like a normal application running on Windows, however it is the browser running inside Kali. This should work for all GUI apps you may run.

If you restart your PC make sure to start up the X Server again.

## Install a better Terminal

Windows now offers a new app on the store called Terminal that works great for using WSL. The main feature being that you can open multiple tabs and install custom profiles. Follow the steps to install it.

### Install Windows Terminal

Open the Microsoft Store and search for "Windows Terminal", here is a link to the store page: https://www.microsoft.com/store/apps/9n0dx20hk701

Once installed search for terminal in your start menu. I like to pin it to my taskbar for easy access. When you open it you will see that it will open the default profile which is CMD. To open a Kali Linux bash terminal, from the drop down on the tab bar click "kali-linux". A new tab will open with your bash terminal.

### Change the default profile

Open the Terminal settings page by pressing `CTRL+,`

You will see your Kali Linux profile which should look something like:

    "guid": "<YOUR KALI GUID>",
    "hidden": false,
    "name": "kali-linux",
    "source": "Windows.Terminal.Wsl"

Copy the GUID for Kali and paste it in the setting for default profile near the top:  

    "defaultProfile": "<YOUR KALI GUID>",

Now when you open terminal, or open a new tab in terminal it will open a new Kali Linux bash tab, this saves you having to select it from the drop down every time.

### Open the bash prompt in the Kali home directory

You might notice that by default when you open a new bash terminal in the terminal app the current working directory will be your Windows home folder. To chnage this and make it open to your Linux home directory update your profile for Kali Linux in the Terminal settings.

Open the Terminal settings page by pressing `CTRL+,`

Add a ',' at the end of the "source" line, and add the commandline line:

    "guid": "<YOUR KALI GUID>",
    "hidden": false,
    "name": "Kali",
    "source": "Windows.Terminal.Wsl",
    "commandline": "wsl.exe ~ -d kali-linux"

You can also update the name to "Kali" like I have.

## Setup SSH for WSL

### Configure OpenSSH in Kali

In your Kali bash shell reinstall OpenSSH by:  
`sudo apt autoremove openssh-server`  
`sudo apt install openssh-server`

Edit the “/etc/ssh/sshd_config” configuration file:  
`sudo nano /etc/ssh/sshd_config`

    Port 2222
    # 
    #
    PasswordAuthentication yes

Remove unnecessary '#', enter the data and save the file. The reason for changing the port is that the default port 22 might be used by windows.

### Open port 2222 in the Windows Firewall

In the Windows Start menu, type "WF.msc".
In the "Windows Firewall with Advanced Security" section, click on "Inbound Rules".

Add a new rule for TCP 2222 and allow the connection:
"Actions" > "New Rule"
"What type of rule would you like to create?": "Port"
"Does this rule apply to TCP or UDP?": "TCP"
"Does this rule apply to all local ports or specific local ports": "Specific local ports: 2222"
"What action should be taken when a connection matches the specified conditions?": "Allow the connection"
"When does this rule apply?": -All Boxes-

### Restart SSH Server

In the Kali bash run:  
`sudo service ssh --full-restart`

### Forward SSH TCP packets to WSL

WSL2 changed the way some things worked in the backend. Now to access tour WSL installation over the network we need to forward applicable packets to WSL. For this create a new NETSH rule.

In your Kali bash terminal get the WSL IPV4 address, run:  
`hostname -I`  
Take note of this address.

Open PowerShell as Administrator and run:  
`netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=2222 connectaddress=<WSL IPV4 ADDRESS>`  
Make sure to substitute in your WSL IP address.

If you ever need to remove this rule, open PowerShell as Administrator and run:  
`netsh interface portproxy delete v4tov4 listenport=2222 listenaddress=0.0.0.0`

### Test the connection

From any PC on your local network you should now be abel to SSH into your WSL Kali:  
`ssh <YOUR WSL USERNAME>@<YOUR HOST PC IPV4 ADDRESS>`

### Access via SSH Key

If you don't want to access Kali with the password but rather an SSH key then do the following.

Generate host key:  
`ssh-keygen -A` 

Generate user key:  
`ssh-keygen -t rsa`

Authorize the user:  
`cat .ssh/id_rsa.pub >> .ssh/authorized_keys`  
`chmod 644 .ssh/authorized_keys`

You can now use this key file to access the machine. I'd advise adding a passphrase to the key for security reasons. To SSH with this key file (located in ~/.ssh/):  
`ssh -i id_rsa <YOUR HOST PC IPV4 ADDRESS>`

## Auto-start/services (systemd and snap support)

As I said earlier you may encounter this error:  
`System has not been booted with systemd as init system (PID 1). Can't operate.`  
`Failed to connect to bus: Host is down`  

There are ways around this, however it may not work for you. Because of the risk of doing these workarounds/hacks I will not go through it here, however here are some helpful links:

https://github.com/shayne/wsl2-hacks  
https://github.com/arkane-systems/genie

## WSL2 Useful Tips

### Shutdown WSL

From PowerShell:  
`wsl --shutdown`

### Open Windows Explorer from bash

To open the Windows Explorer at the current working directory, from the bash command line:  
`explorer.exe .`
