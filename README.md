# g2h-camera-mods
 
This camera supports Homekit Secure Video for a reasonable price - but it has a couple of things that needed fixing.
 
These steps modify the G2H camera to enable RTSP, telnet, lock a few things down and not to call home. To undo these changes you should be able to do a factory reset as two full (A/B) images are stored in the SPI flash - though I have not tested this.

# TL;DR
Clone this repo to the root of an SD card.

Insert the card, power on the camera.

login via telnet --> `telnet G2H_IPADDRESS` (user: `root`, pass: `password`)

The camera will reboot when it's done and remove the SD card. Use at your own risk.

# Changing the voice to English
The default Chinese language was fixed with no way to change it (in FW <2.0.9). Which is fine but when you do certain tasks it would be nice to be able to understand what it was doing.

The audio files are stored in `/etc/ch` which are all in Mandarin. In a later version of the firmware the new audio files appeared (in folders   `en`, `es` and `ru`). There is also German, Italian and French in another partition `/customer/voice`.

### Fix No1
Execute the command `set_language en` to change the language to english.
Edit the language to English (en) in `/mnt/config/factory_config.ini` to retain after a factory reset.

### Fix No2
Using a symbolic link to point each file in `/etc/ch/en/` to the corresponding file in the parent directory is an easy way to change the language to english.
 
Note that these other languages only existed after a software update, and you use the Aquara app to do the update. So I copied these from one device to another.

# Swap to international servers
Edit mentions of `lumi.camera.gwagl02` to `lumi.camera.gwag03`

`vi /mnt/config/miio/device.conf`

`vi /mnt/config/hostapd.conf`

`vi /etc/build.prop`

# Editing Camera Settings via Telnet
`vi /mnt/config/flash_config.ini`

Highlight the value you want to change, press `r` on your keyboard and then the number you want to swap to.

Flip image: under `[video]` change `flip` value to 3
Timestamp: change `OSD` to `1`

Scroll to the bottom, press `i` on the keyboard and delete the line with the checksum

press `esc`, then type `:wq`

generate a new checksum using `md5sum /mnt/config/flash_config.ini`

Open the file again: `vi /mnt/config/flash_config.ini`
At the bottom press `i` on your keyboard and add a new line `checksum = your MD5HASH`
 
# Do not call home
 
The binaries include a bunch of chinese IPs and domain names - I am not sure if they are used when Homekit only mode is enabled (my 5 minute from boot traffic traces indicated they did not). But I removed them anyway, just in case, simply by patching the strings in the binary to point variations of home _127.0.0.1_ and pointing these domain names to localhost also. If you don't want to do this you could also `route <dst> lo` to send the traffic to a black hole.
 
 
```bash
# /etc/hosts
127.0.0.1 cm.iotcplatform.com gm.iotcplatform.com aiot-coap-test.aqara.cn
```
 
The binary also referenced some public DNS servers, these were changed to fake addresses.
 
One of these must have been used for NTP, so manually adding NTP to the boot process solves this.
 
```bash
homekit_ntp au.pool.ntp.org
``
