# Abstract

This is just some quick documentation about how to enable VNC on a new Raspberry Pi installation

1. From the commandline, run `sudo raspi-config`

2. Choose option "3" - `Interface Options`

3. Choose option "I2" - `VNC`

4. Answer `Yes` to "Would you like the VNC Server to be enabled?"

5. After a moment, you should receive a message saying "The VNC Server is enabled"

6. Go back to the main menu and choose `Finish`

7. Configuration file is `/etc/wayvnc/config` - configure your VNC client as appropriate

8. **DO NOT LEAVE** it this way, but if you are struggling to authenticate with your VNC client, you can set both `enable_auth` and `enable_pam` to false

9. Profit?  :)





