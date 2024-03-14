# Abstract

This writeup is intended to provide a setup on a Raspberry Pi4/Pi5 for VS Code Server.  This is specifically written up with a focus on using the setup from an iPad or iPhone (my particular use case is to directly connect the Pi4/5 to an iPad Pro over a USB-C to USB-C cable using the guide located [here] (https://github.com/verxion/RaspberryPi/blob/main/Pi5-ethernet-and-power-over-usbc.md)).

These steps are current as of 2024-03-13 and were validated on a fresh pi5 installation.

# Process

These steps start with the official [Code-Server Site] (https://github.com/coder/code-server?tab=readme-ov-file) and then continue with some steps from a video about [VSCode on iPad Pro - Full Setup Guide with Raspberry Pi] (https://www.youtube.com/watch?v=11YfaGi0Fpk).

1. Run `sudo apt update` to find out if there are any packages that need to be upgraded

2. Run `sudo apt-get upgrade` to upgrade those packages so you are "current"

3. Run `sudo apt install nodejs` to install nodejs, which is a requirement for the VS Code install

4. Run `sudo apt install npm` to install npm, which is also required

5. Run `sudo curl -fsSL https://code-server.dev/install.sh | sh` to install Code Server

6. Run `sudo systemctl enable --now code-server@<user>` (where `<user>` is the user that will be using code server) to have systemd start code-server now and automatically restart it on bootup.

7. Edit (using vi or nano) the file `~/.config/code-server/config.yaml` (note that this should be in `/home/<user>`, NOT `/root`...

8. Change the `bind-addr` from `127.0.0.1` to `0.0.0.0` (to allow access from any host on your network)

9. Change your `password` from the random string of characters to something you choose (this won't be used until after step 18 below)

10. Change `cert` from `false` to `true`.

11. Finally, on the line below `cert`, add a new line containing the following text: `cert-host: <hostname>.local` where `<hostname>` is the hostname of your pi4.

12. Stop `code-server` by issuing the following command: `systemctl stop code-server@<user>` (where `<user>` is the user that will be using code server)

13. Start `code-server` to read in these changes by issuing the following command: `systemctl start code-server@<user>` (where `<user>` is the user that will be using code server)

14. Launch Safari on your iPad and access the following URL: `https://<hostname>.local:8080` where `<hostname>` is your actual pi hostname...

15. You should see a startup screen that says "Get Started with VS Code for the Web".

16. Install Python support or whatever other extensions you wish.  The video linked above was from when apparently you had to do this manually from the commandline.  This issue seems to have been resolved since then, and you will be able to install directly from VS Code's UI.

17. From this page in Safari, tap on the `Share` icon (box with an up arrow coming out of it)

18. Scroll down until you see the option to `Add to Home Screen` (tap this)

19. Change the icon Name to something you like and tap `Add`.  (When you run this home screen "app", it will run full screen)

20. Profit?  :)

# Background

The following are all videos or links to web pages that I referred to before coming up with this guide.

[Quick and Easy Raspberry Pi iPad Setup Guide] (https://www.youtube.com/watch?v=4PAdeZ4aokk)

[Coder Self-Hosted Cloud Development Environments] (https://github.com/coder/coder)

[64 bit CLI version of Visual Studio Code] (https://code.visualstudio.com/docs/?dv=linuxarm64cli)

[VS Code "Code Server" Install] (https://github.com/coder/code-server)

[VSCode on iPad Pro - Full Setup Guide with Raspberry Pi] (https://www.youtube.com/watch?si=im4gdFCbbW-Su8iq&v=11YfaGi0Fpk&feature=youtu.be)

[Intro to Docker using a Raspberry Pi 4] (https://www.youtube.com/watch?v=nBwJm0onzeo)

[How to Create a Great Local Python Development Environment with Docker] (https://www.youtube.com/watch?v=nBwJm0onzeo)

[Use Raspberry Pi 4 USB-C data connection to connect with iPad Pro] (https://magpi.raspberrypi.com/articles/connect-raspberry-pi-4-to-ipad-pro-with-a-usb-c-cable)









