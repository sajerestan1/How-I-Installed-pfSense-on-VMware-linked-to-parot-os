# How I Installed pfSense on VMware and Connected It to Parrot OS: A Personal Journey

If you're like me, curious about firewalls and home labs, you might try setting up pfSense on VMware and connecting it to a Linux VM like Parrot OS. Sounds simple, right? That’s what I thought—until I actually tried it. This write-up captures my journey, mistakes, lessons, and eventually, success.
Step 1: Installing pfSense on VMware (The Easy Start)
1.1 Downloading pfSense

I downloaded the pfSense ISO from the official site and had it ready for installation.

https://www.pfsense.org/download/

1.2 Creating the pfSense Virtual Machine

![1](https://github.com/user-attachments/assets/bda903bd-bdb1-415f-ae75-f811b32f5300)


![2](https://github.com/user-attachments/assets/a74d53ac-b2a7-46c7-984c-7ec4b306e460)

![3](https://github.com/user-attachments/assets/3931d826-9b4b-4422-97e4-a8fd2405bfb5)


![4](https://github.com/user-attachments/assets/16c404f7-360a-44a7-97ea-163aafbc7086)


![5](https://github.com/user-attachments/assets/1c8cfae2-16d9-4841-b260-a67c1b14794e)


![6]![7](https://github.com/user-attachments/assets/9c9942f2-3724-498e-94a5-a036ed83ffb7)





Here's something I learned that made things easier: VMware Workstation automatically detects that you're installing FreeBSD as soon as you select the pfSense ISO via the “Browse” option during VM creation. No need to stress about selecting the OS type manually—it does that for you.

I proceeded with the defaults:

  2 GB RAM

  10–20 GB disk

  Three network adapters:

  WAN on NAT (or Bridged if you prefer that) 

  LAN on a Custom network (VMnet1)
  
  OPT on custom network   (VMnet2) (This is VMnet8 for me but the process is the same.)

 Click the Add Network button in the VMware Network Editor to create the VMnet adapters you'll assign to pfSense.
  
![7](https://github.com/user-attachments/assets/d1a68e73-63e8-4a44-9d0b-a0dafcce8f0a)

![8](https://github.com/user-attachments/assets/8277c91f-8585-45dd-a0e9-a957c3c71a2d)


## Installation Pfsense on VMware
Accept the Copyright and Trademark Notices.

![image](https://github.com/user-attachments/assets/06edfa73-3882-418f-90c9-c156d4720482)

Select "install".

![image](https://github.com/user-attachments/assets/86f66841-4468-4fad-89d1-77afddd7e290)

-Select your keymap

-Use the default partition (we're in a lab, we just need a firewall)

-Proceed with installation (no miror, no encrypt, nothing)

-Wait until the installation is complete

![image](https://github.com/user-attachments/assets/163b2aad-93bc-447f-978f-f82fa02b1f3d)

Do not load manual configuration.

![image](https://github.com/user-attachments/assets/7b783763-0ccd-4451-8cc0-5b3f9f178b78)


### Reboot on the new system.

Configuration
Now, the server is powered on with the new system. 

![image](https://github.com/user-attachments/assets/570231a3-95ba-4939-b855-05442d3b4322)

Select 1 to configure interfaces.

![image](https://github.com/user-attachments/assets/c241c3b4-d42e-4e69-866f-617a050d0306)

I can see the MAC Addresses. Remember,  I made a note of the MAC addresses. It is always helpful to have/know them.

-Select the VMnet0 INTERFACE for WAN

-Select the VMnet1 INTERFACE for LAN

-Select the VMnet2 (VMnet8 in my case) for OPT


![image](https://github.com/user-attachments/assets/a91290fb-5536-4899-8ab0-78d55657c90a)

Confirm the set up

![image](https://github.com/user-attachments/assets/94a9dc6f-dcc0-4d88-9b4d-392d995dd53a)



## Setting Up VMnet in VMware Network Editor

Let me walk you through configuring VMnet networks in VMware:

  Open Virtual Network Editor
  
  Click Edit (top left in VMware Workstation) → select Virtual Network Editor.

  ![10](https://github.com/user-attachments/assets/aba1d3f1-ee76-4be4-85ec-bf047f196905)



  Add and Configure Networks:

  ### VMnet0 (NAT - for WAN):

  Set to NAT

  Enable DHCP

  Check Connect a host virtual adapter to this network

  Leave IP address blank (it will auto-assign)

  Set MTU to 1500 if prompted

  ![11](https://github.com/user-attachments/assets/4d5524ec-9df4-4edf-bbeb-19a257bd3982)



  ### VMnet1 (LAN - pfSense LAN):

  Set a static IP (e.g., 192.168.1.0)

  Uncheck DHCP

  Uncheck Connect a host virtual adapter

  Make sure the subnet mask ends in .0 (e.g., 255.255.255.0)

  ![12](https://github.com/user-attachments/assets/6fbc81db-9e80-478b-b5aa-98d83d924f76)

  ###VMnet2 (OPT - pfSense OPT):

  Set a static IP (e.g., 192.168.2.0)

  Uncheck DHCP

  Check Connect a host virtual adapter

  Ensure subnet mask is correct (255.255.255.0)

  
![13](https://github.com/user-attachments/assets/f615c3d7-f264-42d1-aa2e-5188f5810f91)

  
Step 2: Setting Up the LAN Interface and Connecting Parrot OS
2.1 Installing pfSense

The installation was smooth until I had to configure the interfaces. I skipped the auto-detect the first time and just guessed. Bad move.

Eventually, I let pfSense auto-detect:

    NAT/Bridged interface = WAN

    VMnet1 (custom/internal) = LAN

pfSense set its LAN IP to 192.168.1.1 by default. I enabled DHCP on the LAN interface.
2.2 Setting Up Parrot OS

This was a sneaky challenge. I added Parrot OS as another VM and connected its network adapter to VMnet1, just like pfSense’s LAN. However, when I booted it, ip a showed no IPv4 address—only the IPv6 link-local.

After trying sudo dhclient ens33 and still getting nothing, I manually assigned:

sudo ip addr add 192.168.1.10/24 dev ens33
sudo ip route add default via 192.168.1.1

Boom—Parrot could now ping pfSense’s LAN interface!



### Fixing Network Issues on Parrot OS When Connected to pfSense

At first, I noticed that my Parrot OS had no internet connection and couldn’t ping the pfSense LAN IP (192.168.1.1). Even more confusing — Parrot wasn’t showing any IPv4 address when I ran ip a.

To work around this, I had to manually run the following commands every time I booted Parrot:

    sudo ip addr add 192.168.1.10/24 dev ens33
    sudo ip route add default via 192.168.1.1

This temporarily fixed the problem, but I needed a more permanent solution. After some digging, I found two methods to make the IP configuration persistent:
1. 🧪 Command-line method (nmcli)

This worked well for me:

    nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.10/24
    nmcli con mod "Wired connection 1" ipv4.gateway 192.168.1.1
    nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8"
    nmcli con mod "Wired connection 1" ipv4.method manual
    nmcli con up "Wired connection 1"

After doing this, Parrot was able to connect to pfSense and route its internet traffic through it seamlessly.

2. 🖥️ Graphical method (NetworkManager GUI)

Alternatively, you can do this through the GUI:

    Click the network icon in the taskbar.

    Go to Wired Settings > click the ⚙️ next to “Wired connection 1”.

    Under the IPv4 tab, do the following:

        Method: Manual

        Address: 192.168.1.10

        Netmask: 255.255.255.0

        Gateway: 192.168.1.1

        DNS: 8.8.8.8 or 1.1.1.1

Click Apply and reconnect.

I went with the nmcli method, and it worked like a charm. Parrot now boots up with the right IP address, connects to pfSense, and has internet access routed through the firewall — no more manual commands every time!
