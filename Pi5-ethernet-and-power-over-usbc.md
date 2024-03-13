# Abstract

This writeup explains how to configure your pi5 (from scratch) to allow ethernet over a USB-C to USB-C cable.  This allows you (for example) to plug your pi5 directly into a USB-C iPad or iPhone or Mac to both power it as well as connect to it over SSH **WITHOUT** requiring wifi.

# Process

As of 2024-03-13, here are the steps to get everything working from "scratch":

1. Download and run the "Raspberry Pi Imager" (version at the time of this writing is v1.8.5)

### 2. Raspberry Pi Device: Raspberry Pi 5, Operating System: Raspberry Pi OS (64-bit), Storage: Micro SD card

### 3. OS Customization: Edit Settings

### 4. Set your hostname, username and password, wireless LAN, and Locale settings

### 5. Choose "Yes" to apply your OS customization settings

### 6. Confirm you want to continue (assuming it is correctly listing your micro SD card) by clicking "Yes"

### 7. Wait for the write and the verification to finish

### 8. Plug the micro SD card into your pi5 and then use a high quality USB-C to USB-C cable to connect it to your iPad Pro (or other USB-C iOS device)

```
Note - I used this cable, which I quite like because it is very short, but also very fast and capable of charging at high rates of speed:

[OWC Thunderbolt 4 Cable:] (https://www.amazon.com/dp/B0BLXLSDN5?ref=ppx_pop_mob_ap_share)
```

### 9. Using an iOS app like "Blink" (or truly ANY SSH app), ssh into your pi5 using your previously (step 4) wifi and username/password settings.

```
Note #1 - For the uninitiated, you can use the hostname you set up in step 4 above postpended with ".local".  So if the hostname you configured was "pi5", you would type "ssh username@pi5.local" (where username is whatever you provided in step #4 above) in "Blink" to connect...
  
Note #2 - Your initial connection will need you to trust the host key.  So beware that this is a one time aspect of this connection...
```

### 10. Type "sudo bash" so that you can have elevated privileges for the remaining steps in this guide

```
Note - Alternatively you can pre-pend "sudo " to the front of all the commands in this guide
```

### 11. Edit (this guide assumes you are familiar with either vi or nano) the file /boot/firmware/cmdline.txt and add the following just AFTER "rootwait":

```
modules-load=dwc2,g_ether
```

### 12. Edit (again, pick either vi or nano) the file /boot/firmware/config.txt and confirm it has an UNCOMMENTED line near the end that is "otg_mode=1".  Then add below the line "[all]" (it HAS to be after the "[all]" line for it to work properly) the following:

```
dtoverlay=dwc2
```

### 13. Now we need to add our connection name by running the following command:

```
nmcli con add type ethernet con-name ethernet-usb0
```
    
### 14. Now edit the file we just created (using vi or nano) named /etc/NetworkManager/system-connections/ethernet-usb0.nmconnection

- Note - You will be adding the lines for autoconnect and interface-name, and then modifying the line with "method=" to change auto to "shared":

```
[connection]
id=ethernet-usb0
uuid=<random group of characters here>
type=ethernet
autoconnect=true
interface-name=usb0
  
[ethernet]
  
[ipv4]
method=shared
  
[ipv6]
addr-gen-mode=default
method=auto
  
[proxy]
```

### 15. Create yet another new file (again with vi or nano) at /usr/local/sbin/usb-gadget.sh with the following contents:

```
#!/bin/bash

nmcli con up ethernet-usb0
```

### 16. Make this new file executable with the following command:

```
chmod a+rx /usr/local/sbin/usb-gadget.sh
```

### 17. Create your last new file (using vi or nano) named /lib/systemd/system/usbgadget.service with the following text:

```
[Unit]
Description=My USB gadget
After=NetworkManager.service
Wants=NetworkManager.service
  
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/sbin/usb-gadget.sh
  
[Install]
WantedBy=sysinit.target
```

### 18. Run the following command to enable this new service:

```
systemctl enable usbgadget.service
```

### 19. Reboot your pi5 so all your changes can be put to use.  Run the following command to reboot it:

```
shutdown -r now
```

### 20. To make sure everything went as it should, on your iPad or iPhone, go into Settings and confirm that you an "Ethernet" device listed underneath "Wi-Fi" in settings.  Tap on it and make sure you have an assigned "IP Address" (ie, it isn't blank).  If you do not have an assigned IP address, then something went wrong in the steps above - double check **EVERYTHING**!

### 21. To test that this is working as expected, you can do the following:

  a. Turn Airplane mode on (and confirm you don't have wifi enabled) to ensure you could only be using the USB-C to USB-C cable connected to your pi5 for the following steps
  
  b. From "Blink" (or other iOS ssh app), connect to "<username>@<hostname>.local" where <username> and <hostname> are whatever you configured your pi5 with in step #4 above
  
  c. Confirm you can successfully connect.  If you can connect, then this means you can now power AND connect to your pi5 over a simple USB-C to USB-C cable, no Wi-Fi required!
  
  d. Profit?   :)

# Background Info

After "much success" with my pi4, I bought a pi5 and expected things to be more performant but just as smooth to set up.  I couldn't have been more wrong....

EVERYTHING about my USB-C power+ethernet setup that had worked on my pi4 failed utterly on my pi5.

One of the best writeups I found was this one, but it did not work (it never got the iPad Pro side configured): 

https://www.hardill.me.uk/wordpress/2023/12/23/pi5-usb-c-gadget/

It referenced this issue comment, which I'd already discovered beforehand:

https://github.com/raspberrypi/bookworm-feedback/issues/77#issuecomment-1875575543

I'd also tried to learn and get help from these links, to no avail:

https://github.com/raspberrypi/bookworm-feedback/issues/77

https://github.com/raspberrypi/linux/issues/5737

https://forums.raspberrypi.com/viewtopic.php?t=357938

https://forums.raspberrypi.com/viewtopic.php?t=358527

I provide these links to explain that while there have been a lot of discussions on this topic, none of them worked for me.

I eventually made a frankenstein configuration where I used a USB-C battery to power a USB-C hub, plugged my pi5 into that hub for power, and also plugged an ethernet cable from the hub to the pi5 and the hub directly into my iPad Pro.  This... worked.  But yeah, not a great experience, and quite unwieldy.

Out of desperation, I went to a community of people that know all sorts of great things about iPads and iPhones - the "Blink Discord" server.  That's where I was lucky enough to find "rrgeorge" and "peterb4".  "peterb4" ( https://github.com/gernb ) was the one that ultimately got me up and running.  He had previously gotten his pi5 to work over USB-C cable to his iPad Pro and he graciously answered a barrage of questions as I quizzed him on his setup.  He'd forgotten what he'd specifically done to get his setup working, but by answering my questions, he was able to let me reverse engineer his setup.  Once I got everything working on my own pi5, I took a fresh micro SD card, re-imaged from scratch, and painstakingly put together this guide in the hopes that someone else might benefit from the process being documented.  I did let "peterb4" know, however, that as much as I sincerely hope this guide WILL help others... I am also making it for myself so I can get it all working again if I need to (my feeble mind isn't able to remember things as flawlessly as it once did.  :)







