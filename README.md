Forked from great tutorial of jmstriegel

# Installing 32bit Linux on a 2006 iMac

I have a vintage 17" iMac 4,1 (32-bit CoreDuo / 32-bit EFI) from 2006. These first generation Intel iMacs were absolutely lovely, and
have the following specs:

- 2.0 GHz Intel Core Duo 32-bit processor
- 3GB DDR2 SDRAM
- 1680x1050 TFT Display
- Radeon Mobility X1600 accelerated graphics with 128MB GDDR3 RAM
- Built-in Ethernet and 802.11 wireless
- Looks super cool



## A Few Quirks

There a few things about this machine that make installing Linux difficult, but I've been able to get everything working.
Here's an overview of all the issues I came across.



### 32-bit UEFI 
The iMac's UEFI software also supports a legacy boot mode that presents a BIOS to the OS. I think this was originally intended for running Windows on the iMac in 2006. This happens to be particularly useful for Linux users in 2022!



### Problems with Radeon legacy video driver in EFI vs legacy boot.

The (ancient and unsupported) legacy driver for the Radeon video card does not work properly when booted from EFI on modern kernel versions, requiring the "nomodeset" kernel parameter. This not only disables video acceleration but causes the card to run without power management (hot!), and even breaks sleep functionality. It also makes desktop UI and video playback absurdly slow.

I could be mistaken, but I'm almost positive that many years ago, perhaps around 2015 with a much older kernel, I had hardware video acceleration working in EFI mode on this machine. This is no longer the case in 2022. I have tried getting the legacy Radeon driver to work under EFI boot on both 64-bit and 32-bit kernels and have been unsuccessful. I've pored over countless forums, tried numerous hacks, and have come to the conclusion that this is an unsolved issue. If you want to boot a 2006-07 iMac from EFI, you will have to disable video acceleration using the *nomodeset* kernel parameter or you will have to track down and fix an ancient bug in the legacy radeon driver in the Linux kernel.

**BUT!**

If you want video acceleration, sleep/suspend, and power throttling to work properly on the iMac5,1 with 64-bit Linux, there is at lest one way that works: boot in legacy BIOS mode with Grub in the MBR. The legacy Radeon firmware properly identifies the device and with a few configuration settings everything appears to work quite well.

| Boot Method           | OS             | Radeon driver
| --------------------- | -------------- | -----------------
| EFI (rEFInd 32)       | Linux (32 bit) | unaccelerated (nomodeset)
| EFI (Grub EFI 32)     | Linux (32 bit) | unaccelerated (nomodeset)
| Legacy (Grub in MBR)  | Linux (32 bit) | works! (over refind)


### Should I dual boot?

You can do with Snow Leopard and refind. I  would suggest to do it, just to install refind and have a base to go back to, if everything goes bonkers. You need a Snow Leopard DVD.
I recommend purchasing an affordable SATA SSD and replacing the old hard drive. A 500GB Crucial SATA SSD is cheap, everything will run MUCH faster, and you won't have to worry about accidentally losing something important from your old disk. An external USB enclosure for the old drive will allow you to access all the files from your original setup.



### Linux installer ISO issues

Getting a installer ISO to boot on this machine has become more complicated over the years. USB boot doesnt work natively, so you you need refind, but only works in legacy mode.

Debian has a ["multi-arch" ISO](https://cdimage.debian.org/debian-cd/current/multi-arch/iso-cd/
) that contains both 32-bit and 64-bit EFI images. This is both official and current (though maybe I should be concerned that it's not linked from the downloads page)... I'll be using this and preparing a USB thumb drive from another computer.


### 802.11 WiFi drivers during installation

The WiFi driver firmware is closed-source and not available on the official install media. The simplest option is to use ethernet during install and install the 802.11 drivers afterward.



### iSight camera firmware (important!)

The only way to get the iSight working is to lift the firmware off a MacOS installation (though you can probably find the file online somewhere). Details are below, but you'll want to make sure you have a copy of the driver before wiping your old hard disk. In my case, I've replaced my OS X drive with an SSD. This has the additional benefit that I can access all of my old files from the old disk, if necessary.




## Installation HOWTO

With all that out of the way, here are the complete instructions for welcoming your 2006 iMac into the year 2022.


### Back up your iSight Firmware

Before you do anything else, copy the proprietary iSight firmware to a safe place. You'll need this file after you install Linux in order to get the camera working. It's on your iMac's drive at the following location:

```
/System/Library/Extensions/IOUSBFamily.kext/Contents/PlugIns/AppleUSBVideoSupport.kext/Contents/MacOS/AppleUSBVideoSupport
```

Should you miss this step, you'll probably have luck finding a copy of this file online. That said, I worry about some of those links eventually disappearing from the internet, so best not to skip this step.



### Upgrade to an SSD (or backup your HD)

We're going to reformat whatever's in your machine, so back up your iMac's drive -- or better, replace it with an SSD and keep the original as a backup.

I recommend replacing the original ATA hard disk with a new SATA SSD for performance reasons alone. A Crucial MX500 is a good choice. It's currently under $70 on Amazon and probably cheaper when you read this. You'll also need a 2.5" to 3.5" adapter to mount it in the old drive's space.

[Crucial MX500 SATA SSD](https://www.amazon.com/Crucial-MX500-500GB-NAND-Internal/dp/B0784SLQM6/)

[SATA 2.5" to 3.5" Adapter](https://www.amazon.com/General-Drive-HDD-Adapter-CADDY/dp/B00F3QFKNS/)

iFixit has a decent [guide for replacing the drive](https://www.ifixit.com/Guide/iMac+Intel+20-Inch+EMC+2105+and+2118+Hard+Drive+Replacement/1092). I think they also sell the necessary tools if you don't have them.  You'll need a long, slender driver and variously sized torx bits.


**Note: it's really hard getting the case off.** There are hidden latches at the top that you're supposed to be able to depress by pushing a plastic card into a slot at the back of the machine. It took me a very long time to finally get the latch to disengage. Take your time and be patient -- it'll eventually come free. I believe there's a less invasive way to pull the latches using a strong neodymium magnet, but I don't have one and can't confirm it works. It's worth a shot if you have a strong magnet, because the card-latch trick is a real pain.

**Note 2:** take this opportunity to use some compressed air and a vacuum to thourougly clean things out. The computer is a decade and half old and full of dust bunnies. This will help with ventilation and can prevent overheating and premature component failure.

You can also buy a cheap external enclosure to access files on the old disk via USB.

[External 3.5" SATA USB Enclosure](https://www.amazon.com/Inateck-Inch-Enclosure-Support-SA01003/dp/B07N63MXN6/)




### Boot media

Download the latest Debian multi-arch network install ISO here (currently, debian-11.4.0-amd64-i386-netinst.iso "Bullseye"):
[https://cdimage.debian.org/debian-cd/current/multi-arch/iso-cd/](https://cdimage.debian.org/debian-cd/current/multi-arch/iso-cd/)

The multi-arch network install ISO (amd64-i386-netinst) should have a 32-bit EFI boot image and (unlike the i386 version) it will install the 64-bit kernel and user-space software.

Use your favorite disk imaging method or just use the unix 'dd' to write the ISO to a USB stick.

**NOTE: make sure to replace sdX with you actual usb device. Be careful not to accidentally use the device for your computer's main drive!**

```
dd if=debian-11.4.0-amd64-i386-netinst.iso of=/dev/sdX bs=1M status=progress
```

## install snow leopard through dvd, create bootcamp partition and install refind 
Install Snow Leopard, create a bootcamp partition. Decrease the size of the bootcamp partition with disk utility to allow small partition for swap space for linux (5 Gb should do). 
Install refind.
Boot your iMac with the USB drive plugged in. It should appear as an EFI bootable removable disk in refind. Choose legacy. Grub will load and you can begin the Debian installer.


### Ethernet is required during installation

The network installer requires internet access to download and install packages. However, drivers for WiFi aren't installed yet. You can install these from USB somehow, but I'd recommend just plugging your computer into ethernet during this portion of installation.

Connect to ethernet, and tell Debian to ignore/skip any proprietary drivers during installation.

It's proabably also advisable to use a USB mouse until after installation when the wireless drivers are known to be working.



### Formatting the new drive for legacy boot
Start the installer of debian. When reached partition choose manual and create two partitions: 
1. format bootcamp partition to hold /
2. format swap partition to swap (huhhhu)

### Complete installation and reboot to Grub

AT THE END OF THE INSTALLATION, MAKE SURE TO MANUALLY ENTER /dev/sda3 AS LOCATION TO INSTALL GRUB

Reboot and remove the USB drive. When the iMac chimes, hold down the option key. You should see a hard disk icon appear, with the strange word "Windows" written beneath. Close your eyes, imagine "Linux" and hit enter to boot into the Grub menu.

### First boot and fixing a boot options

At the Grub menu, press "e" to edit the boot parameters and add the "nomodeset" kernel parameter to the linux line. This will likely be needed on the first boot until a couple of changes are made in the next steps.

#### Give your user account sudo access

Log in with your user account, and run "su -" in a terminal window to become the root user. Run the following to give your user account sudo permissions:

```
adduser YOURUSERNAME sudo
```

Later, when you're done with installing everything and you've confirmed sudo is working, you might consider disabling login on the root account with:

```
sudo passwd -l root
```

#### Disable Wayland

Wayland has very serious glitching issues with this video hardware once acceleration is working. Oddly, the mouse cursor appears fine, but the background textures in gdm and other user interface elements are glitchy and smeared about the screen. There must be something Wayland is using that is not supported by the hardware. XOrg runs almost perfectly, however, so you can just disable Wayland and pretend you didn't see this.

Edit /etc/gmd3/daemon.conf and remove the comment from the following line:

```
WaylandEnable=false
```

#### Install nonfree firmware and drivers for Radeon Mobilty X1600 and b43 802.11 wireless.

Add the non-free and contrib sources to the apt package manager. You can do this by checking a couple boxes in the "Software and Updates" application, or you can edit /etc/apt/sources.list and add "non-free contrib" at the end of every deb and deb-src line (after "main"). It'll look something like this:

```
deb http://deb.debian.org/debian/ bullseye main non-free contrib
deb-src http://deb.debian.org/debian/ bullseye main non-free contrib

deb http://security.debian.org/debian-security bullseye-security main non-free contrib
deb-src http://security.debian.org/debian-security bullseye-security main non-free contrib

deb http://deb.debian.org/debian/ bullseye-updates main non-free contrib
deb-src http://deb.debian.org/debian/ bullseye-updates main non-free contrib
```

Save and then update your packages:

```
apt-get update
apt-get dist-upgrade
```

Finally, we can install the video driver firmware and wireless drivers.

```
apt-get install firmware-amd-graphics
apt-get install firmware-b43-installer
```


#### Fix kernel boot parameters

Now edit /etc/default/grub and change the GRUB_CMDLINE_LINUX_DEFAULT to add "radeon.dpm=1 radeon.modeset=1 radeon.pcie_gen2=0". Mine looks like this:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet radeon.dpm=1 radeon.modeset=1 radeon.pcie_gen2=0"
```
What those are for:

- radeon.modeset=1 enables kernel modesetting on the video card, essentially the opposite of nomodeset (and probably enabled by default if things are working).
- radeon.dpm=1 ensures that display power management is running.
- radeon.pcie_gen2=0 this one forces the card to run in pcie gen1 mode. Without this setting I had numerous video compositing and text glitches in gnome.

I don't much care for the background image at boot. You can disable it by adding the line:

```
GRUB_BACKGROUND=""
```

When you're done changing things, you need to run:

```
sudo grub-update
```


#### Install the iSight firmware

Copy the AppleUSBVideoSupport file (that you saved earlier) to your home directory. The following command with install the isight-firware-tools, which will extract the firmware from this file and install it to the correct place.

```
sudo apt-get install isight-firmware-tools
```

During installation, it will prompt you for the location of AppleUSBVideoSupport. Change this to /home/YOURUSERNAME/AppleUSBVideoSupport (or to whatever location you placed the file).  If you make a mistake, you can run `sudo ift-extract -a [path_to_AppleUSBVideoSupport]` with the correct path.

After reboot, you can use the pre-installed "Cheese" program to test your camera. Frankly, the camera quality is not that great. Ah well.


#### Reboot

Reboot your iMac. Refind should start and youn should be able to choose legacy debian (one of three in my bootmenu) and almost everything should be working now. 
Reboot again and cleanup refind bootmenu by choosing bootentry you are NOT using and choose '-'. Refind asks you to hide it. Maybe you should go to refind.conf and uncomment showtools hidden_tags

Here are some things to confirm:

  - Xorg should have booted properly. If the screen looks really corrupted, see above about disabling Wayland.
  - Run `sudo dmesg |grep drm` and confirm the accelerated video card driver is now functioning. You should see a message about modesetting being enabled, as well as a bunch of other information about the graphics card (detected as an RV530).
  - WiFi should be working. You can unplug from Ethernet now and configure your WiFi settings.
  - Bluetooth should be working. Set up your wireless mouse, etc.
  - Suspend should work correctly. The light on the iMac will "breathe" when the machine is in suspend mode and hitting the power button once should reanimate everything.

**NOTE: if you forgot to disable Wayland** your screen is probably a glitchy mess at this point. You'll need to switch to a text console with ctl-alt-F3 to get anything done. Wayland glitches when video acceleration is finally working on this hardware, and the screen will be incomprehensible. See disabling Wayland above.



## UI Theming HOWTO (optional)

The default theme is perfect for my other computers, but the 2006 iMac is particularly enjoyable with a few updates.

### Install gnome tweaks and the dash to dock extension

```
apt-get install git gnome-tweaks gnome-shell-extension-dashtodock
```

### Install WhiteSur-gtk-theme

There are a lot of options but the defaults are a pretty nice BigSur-like theme for gnome. More details and examples are on the author's github: https://github.com/vinceliuice/WhiteSur-gtk-theme

```
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git
cd WhiteSur-gtk-theme
sudo ./install.sh
```

You can similarly tweak the look of Firefox, the dash to dock extension, and the gdm login screen:

```
sudo ./tweaks.sh -f -g -d
```


### Install WhiteSur-icon-theme WhiteSur-wallpapers and WhiteSur-cursors

Just a few other tweaks from the same theme author. These make your icons and backgrounds snazzy.

Icons:

```
git clone https://github.com/vinceliuice/WhiteSur-icon-theme.git
cd WhiteSur-icon-theme
sudo ./install.sh
```

Desktop and login backgrounds:

```
git clone https://github.com/vinceliuice/WhiteSur-wallpapers.git
cd WhiteSur-wallpapers
sudo install-wallpapers.sh
```

Cursors:

```
git clone https://github.com/vinceliuice/WhiteSur-cursors.git
cd WhiteSur-cursors
sudo install.sh
```


### Configure everything in Gnome Tweaks

  - Enable extensions.
  - In extensions, enable Dash to dock
      - Position on bottom
      - Set to use default theme
  - In appearance settings, select:
      - Themes: WhiteSur-light
      - Cursor: WhiteSur-cursors
      - Icons: WhiteSur
      - Background: /usr/share/backgrounds/WhiteSur-light.jpg
      - Lock Screen Image: /usr/share/backgrounds/WhiteSur-light.jpg
  - Window Titlebars:
      - Enable Maximize and Minimize buttons
      - Placement: Left

You might need to reboot before all of these changes take effect and look consistent.

NOTE: Gnome Tweaks is a little glitchy with left button placement enabled. You need to stretch the window wider every time you run it or it awkwardly collapses the menu content. Fortunately, I haven't seen other apps with this issue.

### Remove the debian logo from gdm (optional)

The gdm logo setting is in /etc/gdm3/greeter.dconf-defaults. Edit this file and look for the "[org/gnome/login-screen]" section. Add the following line to remove the logo from the login screen:

```
logo=''
```


## Cleaning up and have some nive tweaks while booting (files are uploaded in the code repository)

To get a slick minimalistic bootup experience you can skin refind with minimal theme, and install plymouth theme macos inspired with debian logo
refind-minimal:
https://github.com/evanpurkhiser/rEFInd-minimal

Darwin plymouth gives a nice tutorial to get it working. There is a debian logo fork from this.
https://github.com/libredeb/darwin-plymouth
So the code for the debian logo that works in the same way.
https://www.gnome-look.org/p/1888173/

#clean up boot to make sure you get minimal blinking cursors, boot messages and such
your /etc/default/grub should look like this

GRUB_DEFAULT=0
GRUB_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT="0"
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_BACKGROUND=""
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash radeon.dpm=1 radeon.modeset=1 radeon.p>
GRUB_DISABLE_OS_PROBER="true"

```
sudo nano /etc/default/grub
```

Paste the above in the grub
<ctri> <shift> v
<ctrl> x and y

```
sed -i 's/^quiet_boot="0"/quiet_boot="1"/g' /etc/grub.d/10_linux
```
  
  
  
```
sudo update-grub
```

  
https://unix.stackexchange.com/questions/527886/hide-grub-menu-and-initial-loading-message



## Done

That's about everything.  Reboot and enjoy your new 2022 iMac 4,1. Web Browsing, Email, Multimedia -- all the things!

Performance-wise, it's an enjoyable computer in 2022. It's showing it's age in a few places, of course: you might encounter an occasional video driver glitch. Fullscreen youtube is not great. However, Youtube works quite well in a window or theater mode. Aside from these small gripes, it's a fun computer to use. Thanks to the SSD upgrade, many things even seem faster than when this iMac was new.

  
  
  
## Afterthoughts
It took me a while to get to finally get rid of the "loading Grub... Welcome to grub" messages at boot. Seems hard coded in the source of debian. So I uninstalled grub-pc and different other grtub packages. I change /etc/apt/sources.list to ubuntu bionic mirror (temporarily). Reinstalled the ubuntu grub-packages (to sda3!) and apt-mark hold the packages to prevent them from upgrading after I put debian mirror back. Real ugly hack.
Note: grub-shusher didn't work for me.
