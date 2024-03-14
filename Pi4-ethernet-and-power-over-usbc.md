# Abstract

This writeup explains how to configure your pi4 (from scratch) to allow ethernet over a USB-C to USB-C cable.  This allows you (for example) to plug your pi5 directly into a USB-C iPad or iPhone or Mac to both power it as well as connect to it over SSH **WITHOUT** requiring wifi.

# Process

As of 2024-03-13, here are the steps to get everything working from "scratch":

1. Download and run the `Raspberry Pi Imager` (version at the time of this writing is v1.8.5)

2. Raspberry Pi Device: Raspberry Pi 4, Operating System: Raspberry Pi OS (64-bit), Storage: Micro SD card

3. OS Customization: Edit Settings

4. In the General tab, set your hostname, username and password, wireless LAN, and Locale settings.  In Services tab, make sure you enable SSH.

5. Choose `Yes` to apply your OS customization settings

6. Confirm you want to continue (assuming it is correctly listing your micro SD card) by clicking `Yes`

7. Wait for the write and the verification to finish

8. Plug the micro SD card into your pi4 and then use a high quality USB-C to USB-C cable to connect it to your iPad Pro (or other USB-C iOS device)

- I used this cable, which I quite like because it is very short, but also very fast and capable of charging at high rates of speed:

[OWC Thunderbolt 4 Cable:] (https://www.amazon.com/dp/B0BLXLSDN5?ref=ppx_pop_mob_ap_share)

9. Using an iOS app like [Blink] (http://blink.sh) (or truly ANY SSH app), ssh into your pi4 using your previously (step 4) wifi and username/password settings.

- For the uninitiated, you can use the hostname you set up in step 4 above postpended with `.local`.  So if the hostname you configured was `pi4`, you would type `ssh username@pi4.local` (where username is whatever you provided in step #4 above) in [Blink] (http://blink.sh) to connect...
  
- Your initial connection will need you to trust the host key.  So beware that this is a one time aspect of this connection...

10. Type `sudo bash` so that you can have elevated privileges for the remaining steps in this guide

- Alternatively you can pre-pend `sudo ` to the front of all the commands in this guide

11. Edit (this guide assumes you are familiar with either vi or nano) the file `/boot/firmware/cmdline.txt` and add the following just AFTER `rootwait`:

```
modules-load=dwc2,g_ether
```

12. Edit (again, pick either vi or nano) the file `/boot/firmware/config.txt` and confirm it has an UNCOMMENTED line near the end that is `otg_mode=1`.  Then add below the line `[all]` (it HAS to be after the `[all]` line for it to work properly) the following:

```
dtoverlay=dwc2,dr_mode=peripheral
```

13. Edit (again, pick either vi or nano) the file `/etc/modules` and add the following at the end of the file:

```
libcomposite
```

14. Edit (again, pick either vi or nano) the file `/etc/dhcpcd.conf` and add the following at the end of the file:

```
denyinterfaces usb0
```

15. Install `dnsmasq` by running the following command:

```
sudo apt-get install dnsmasq
```

16. Edit (again, pick either vi or nano) the new file `/etc/dnsmasq.d/usb` and add the following text:

```
interface=usb0
dhcp-range=10.55.0.2,10.55.0.6,255.255.255.248,1h
dhcp-option=3
leasefile-ro
```

17. Edit (again, pick either vi or nano) the new file `/etc/network/interfaces.d/usb0` and add the following text:

```
auto usb0
allow-hotplug usb0
iface usb0 inet static
  address 10.55.0.1
  netmask 255.255.255.248
```

18. Edit (again, pick either vi or nano) the new file `/root/usb.sh` and add the following text:

```
#!/bin/bash
cd /sys/kernel/config/usb_gadget/
mkdir -p pi4
cd pi4
echo 0x1d6b > idVendor # Linux Foundation
echo 0x0104 > idProduct # Multifunction Composite Gadget
echo 0x0100 > bcdDevice # v1.0.0
echo 0x0200 > bcdUSB # USB2
echo 0xEF > bDeviceClass
echo 0x02 > bDeviceSubClass
echo 0x01 > bDeviceProtocol
mkdir -p strings/0x409
echo "fedcba9876543211" > strings/0x409/serialnumber
echo "Ben Hardill" > strings/0x409/manufacturer
echo "PI4 USB Device" > strings/0x409/product
mkdir -p configs/c.1/strings/0x409
echo "Config 1: ECM network" > configs/c.1/strings/0x409/configuration
echo 250 > configs/c.1/MaxPower
# Add functions here
# see gadget configurations below
# End functions
mkdir -p functions/ecm.usb0
HOST="00:dc:c8:f7:75:14" # "HostPC"
SELF="00:dd:dc:eb:6d:a1" # "BadUSB"
echo $HOST > functions/ecm.usb0/host_addr
echo $SELF > functions/ecm.usb0/dev_addr
ln -s functions/ecm.usb0 configs/c.1/
udevadm settle -t 5 || :
ls /sys/class/udc > UDC
ifup usb0
service dnsmasq restart
```

19. Make `/root/usb.sh` executable by running the following command:

```
chmod a+rx /root/usb.sh
```

20. Edit `/etc/rc.local` and add this text right before `exit 0`:

```
/root/usb.sh
```

21. Reboot your p4 so all your changes can be put to use.  Run the following command to reboot it:

```
shutdown -r now
```

22. To make sure everything went as it should, on your iPad or iPhone, go into Settings and confirm that you an `Ethernet` device listed underneath `Wi-Fi` in settings.  Tap on it and make sure you have an assigned `IP Address` (ie, it isn't blank).  If you do not have an assigned IP address, then something went wrong in the steps above - double check **EVERYTHING**!

23. To test that this is working as expected, you can do the following:

- Turn Airplane mode on (and confirm you don't have wifi enabled) to ensure you could only be using the USB-C to USB-C cable connected to your pi4 for the following steps
  
- From [Blink] (http://blink.sh) (or other iOS ssh app), connect to `<username>@<hostname>.local` where `<username>` and `<hostname>` are whatever you configured your pi4 with in step #4 above
  
- Confirm you can successfully connect.  If you can connect, then this means you can now power AND connect to your pi4 over a simple USB-C to USB-C cable, no Wi-Fi required!
  
- Profit?   :)

# Background Info

This was the main guide that helped me get my pi4 setup:

[Pi4 USB-C Gadget] (https://www.hardill.me.uk/wordpress/2019/11/02/pi4-usb-c-gadget/)

This video guide was also GREAT for initially being able to connect to my pi4 from my iPad Pro:

[Quick and Easy Raspberry Pi iPad Setup Guide] (https://www.youtube.com/watch?v=4PAdeZ4aokk)



