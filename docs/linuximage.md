---
layout: page
title: Linux Image
permalink: /linuximage/
---
The following instructions walk you through imaging an SD card running Ubuntu 22.04 for a Raspberry Pi. 
This has been tested on both 2GB and 8GB Raspberry Pi 4B computers. 

Last Updated: March 26, 2023

# Installing Ubuntu 22.04
Use the Raspberry Pi Imager to install Ubuntu Server 22.04 LTS (64-BIT)

## Compiling a New Linux Kernel
(ADD EXPLANATION)

Follow the instructions from the [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compiling-the-kernel) to cross-compile a newer version of their linux kernel [rpi-6.1.y](https://github.com/raspberrypi/linux). 
To make sure you are compiling this specific version, swap out their **Get the Kernel Sources** command with 
    
    git clone --depth=1 https://github.com/raspberrypi/linux --branch rpi-6.1.y

*Note this process is time consuming. 
I used a virtual machine running Ubuntu 22.04 and it took roughly an hour to compile. However, this is still far far far faster than attempting to compile on the Pi itself.* 

Once the cross-compilation is complete, insert your SD card into your Pi and boot it up. 
Be sure to run 

    sudo apt update 
    
and then

    sudo apt upgrade 

You will encounter a notification every time you run apt that your kernel is not the expected version and you should reboot. 
This is expected because we are using the rpi-6.1 kernel and Ubuntu 22.04 expects rpi-5.15.
To get rid of this error, you can remove the application that checks to see if you need to restart your system after a new installation. 
However, please do this with caution and reboot your Pi whenever you install applications or services that may impact system operations as apt will no longer run this check for you. 

    sudo apt-get purge needrestart

Finally, I always like to have the option to use `ifconfig` to obtain networking information, but this tool doesn't come pre-installed on the Ubuntu Server. 
Now is a good time to install it by running:

    sudo apt install net-tools

Finally, reboot your Pi before proceeding. 

## Installing a Desktop Environment
After testing both the full Ubuntu desktop as well as lightweight versions xubuntu and lubuntu, I recommend installing xubuntu as it boots without issue and provides a much faster experience than the full desktop. 
I was unable to get lubuntu to boot into the desktop environment, but [this blog](https://waldorf.waveform.org.uk/2020/ubuntu-desktops-on-the-pi.html) that Ubuntu cites highly recommends lubuntu, so do with that what you will. 

For xubuntu, I also tested both the full desktop and the core. I found the core version did everything I needed, and this way I was able to only install additional applications when needed to save space. 

    sudo apt-get install xubuntu-core^

Once the install is complete, reboot your Pi for the changes to take effect. 
Neither xubuntu or xubuntu-core will install a familiar web browser by default. 
I opted to install firefox using:

    sudo apt install firefox


## Installing RealVNC Server 
Since I intend on working with a camera, I find it helpful to have the option to be able use VNC. 
I tested out TigerVNC and RealVNC, but found RealVNC easier to work with in my specific circumstances. 
However, installing the RealVNC Server is not trivial because they don't offer a download for an arm64 version on their website. 
Instead, we'll have to grab the deb package from the Raspberry Pi Archive and install with dpkg. 
You can do this with the following commands. 

    wget http://archive.raspberrypi.org/debian/pool/main/r/realvnc-vnc/realvnc-vnc-server_7.0.1.49073_arm64.deb

    sudo dpkg -i realvnc-vnc-server_7.0.1.49073_arm64.deb

Note that this is the most updated package available at the time of writing. 
You can always check for a more recent release by going to the archive and looking for the latest version number - but make sure you are downloading the *server* and that it is *arm64*. 

Once the RealVNC Server has been successfully installed, we must enable and start the related services to make sure our VNC server is up and running and starts at boot. 

    sudo systemctl enable vncserver-virtuald.service
    sudo systemctl enable vncserver-x11-serviced.service
    sudo systemctl start vncserver-virtuald.service
    sudo systemctl start vncserver-x11-serviced.service

You will know the process has completed successfully if you reboot and see a white and blue VNC icon in the upper right section of your task bar. 
Clicking on this icon will open a panel with your RealVNC connectivity information including your IP address and catchphrase. 

## Setting Up Headless Operation for VNC Use
All should work perfectly now if you're planning on having a monitor attached to your Pi at all times. 
But most of us here are planning on using our Pi as part of a robot. 
This means, in almost all cases, we will not be able to have a monitor attached to our Pi so we have to tell Ubuntu to activate the desktop environment even when there is no monitor attached. 
To do this, we have to make a few modifications to our `/boot/firmware/config.txt` file. 
But first, a disclaimer...

*While I run the risk of being ostracized from the engineering community for saying this, I've become quite fond of `nano` as an editor in terminal. 
I also think that what it lacks in features in makes up for in beginner user-friendliness, and I'm here to teach beginners. 
So, from here on out, all of my instructions for editing files in terminal will use nano. Gasp!*

Moving on...run the following command to open your boot configuration file. 

    sudo nano /boot/firmware/config.txt

This file may look familiar because you should have modified it as part of the kernel business above.
At the end of the file, insert the following three lines:

    hdmi_force_hotplug=1
    framebuffer_width=1920
    framebuffer_height=1080

These lines ignore the status of the hdmi port and set the resolution for your desktop. 
Though the forums seem to debate the reason for needing this particular resolution, I have found it to work without issue so I'm sticking with it.  


Also, look through and see if you have a line that reads `dtoverlay=vc4-kms-v3d`. 
If you do (I didn't on a clean install, but some of my older builds do have this and many tutorials will tell you to add it), comment it out so that line reads `# dtoverlay=vc4-kms-v3d`.

Finally, hit CTRL-X, then y, then enter to save the file. 
Reboot your Pi for the changes to take effect. 

## Fixing the Screensaver
As you use your Pi via VNC, you may notice a strange phenomenon where if you leave the desktop idle for a period of time it stops responding to mouse clicks. 
Your mouse will still move, but you can't actually do anything anymore. 
No matter what settings you change on the built in screensaver, this will continue to happen. 
There is a glitch with the xubuntu screensaver that causes this. 
So, let's install a screensaver that will actually allow you to disable idle sleep and remove the built-in screensaver that is causing all the trouble. 

    sudo apt install XScreensaver
    sudo apt purge xfce4-screensaver

Restart and then look for your screensaver application in your applications menu and within the app disable the screensaver.

BOOM - you now have a fully functional system!
