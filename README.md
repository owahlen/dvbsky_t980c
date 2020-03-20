# Installation instructions for DVBSky T980C card under Ubuntu eoan (19.10)
The
[DVBSky T980C DVB-T2/T/C PCIe with CI](http://www.dvbsky.net/Products_T980C.html)
card is a nice match for watching cable TV on a Linux machine since it is natively supported since kernel 3.19.
It comes with a [CI](https://en.wikipedia.org/wiki/Common_Interface) slot that allows watching of
Pay TV channels.
Even though it is supported by the kernel several installation steps are required that are outlined below.

## Installing the TV Tuner
### Firmware
As outlined on the 
[linuxtv](https://www.linuxtv.org/wiki/index.php/DVBSky_T980C)
page of the card there are two versions that require different firmeware files.
Those files can be downloaded from the 
[OpenELEC firmware collection](https://github.com/OpenELEC/dvb-firmware/tree/master/firmware).

In contrast to what is noted on the linuxtv page the
Version 2 of the card requires the following **two** files to be copied
into the `/lib/firmware` folder:
* [dvb-demod-si2168-b40-01.fw](ressources/dvb-demod-si2168-b40-01.fw)
* [dvb-tuner-si2158-a20-01.fw](ressources/dvb-tuner-si2158-a20-01.fw)

After a reboot the command `dmesg` should contain a line like this:
```
cx23885: CORE cx23885[0]: subsystem: 4254:980c, board: DVBSky T980C [card=46,autodetected]
```

### IR Remote
The card is shipped with an
[infrared remote control](https://www.amazon.de/DVBSky-T980C-DVB-T2-Common-Interface/dp/B01LXOY7BP).
In order to setup this device lirc needs to be installed:
```
sudo apt-get install lirc
```

However, in eoan the lirc package has a bug that leads to a segmentation fault
in the installation process. To solve this, copy the following files into
the folder `/etc/lirc` and re-installing the lirc package.
* [lircd.conf](ressources/lircd.conf)
* [lirc_options.conf](ressources/lirc_options.conf)

The IR controller is disabled by default in the `cx23885` kernel module.
It can be enabled by setting the module option `enable_885_ir` to 1.
This can be achieved permanently by copying the file
[cx23885.conf](ressources/cx23885.conf) to the folder `/etc/motprobe.d`.

Check if the currently logged in user is in the `video`
group with the command `id`.
If this is not the case the user with the following command:
```
sudo usermod -a -G video $(whoami)
```

The 5.3.0 eoan kernel comes with a keymap for the DVBSky IR that maps
scan codes to key codes. The codes do not match exactly the ones of the
remote shipped with the T980C card. This can be solved by replacing
the file `/lib/udev/rc_keymaps/dvbsky.toml` with the following file
[dvbsky.toml](ressources/dvbsky.toml)
After another reboot the IR should be recognized by the kernel.
The commands `sudo ir-keytable -t` and `irw` should return
scan codes and key codes, respectively. 

In order to interact properly with a media player irexec needs to be
configured and started.
For [Kaffeine](https://wiki.ubuntuusers.de/Kaffeine/)
this can be achieved by copying the file
[lircrc](ressources/lircrc) to the folder `~/.config`.
In order to permanently start irexec open the gnome session properties
with the command
```
gnome-session-properties
```
and add an entry with the following properties:
* Name: _irexec_
* Command: _irexec -d_
* Comment: _IR-Remote_

The irexec configuration relies on `qdbus` and `xte` to be installed.
Note that xte is part of the xautomation package.
```
sudo apt-get install qdbus
sudo apt-get install xautomation
```

### Suspend-Resume Issue
After a suspend to ram and subsequent resume there is no TV signal and the IR remote stops working.
This seems to be related to a bug in the cx23885 module.
A workaround is to unload and load the module with a script.
Copy the file [dvbsky-sleep.service](ressources/dvbsky-sleep.service) to the folder
`/etc/systemd/system` and enable the service:
```
sudo cp dvbsky-sleep.service /etc/systemd/system
sudo systemctl enable dvbsky-sleep.service
```
