# PiKVM-HDMI-Passthrough
PiKVM HDMI passthrough on V3 or DIY platforms using CSI-HDMI bridge board (TC358743)

Update your PiKVM software first! 

If you don't have pikvm-update in your software use the documentation here: https://docs.pikvm.org/_update_os/
 


## PiKVM V3:

**Note:** _On the PiKVM V3 HDMI1 is blocked by the fan in the case. You **MUST** recompile to use HDMI0._

### HDMI0 Port Passthrough

### Root Write Mode & Update

```
$ su -
$ Password: root (If you never changed it!?)
$ rw
$ pikvm-update
```

### Git Source

Pull ustreamer source from repo per instruction here: https://github.com/pikvm/ustreamer?tab=readme-ov-file#make

```
$ mkdir code
$ cd code
$ git clone --depth=1 https://github.com/pikvm/ustreamer
```

Edit ustreamer/src/libs/drm/drm.c

Line [84](https://github.com/pikvm/ustreamer/blob/c848756d53626d2ba462a698777c6f4e32bf100c/src/libs/drm/drm.c#L84), replace HDMI-A-2 with HDMI-A-1

```
	drm->path = "/dev/dri/by-path/platform-gpu-card";
	drm->port = "HDMI-A-2"; // OUT2 on PiKVM V4 Plus
	drm->timeout = 5;
```
to 
```
	drm->path = "/dev/dri/by-path/platform-gpu-card";
	drm->port = "HDMI-A-1"; // HDMI0 on PiKVM V3
	drm->timeout = 5;
```

### Compile ustreamer

Directory /root/code/ustreamer/
```
$ make WITH_GPIO=1 WITH_SYSTEMD=1 WITH_JANUS=1 WITH_V4P=1
```

### Install

Install and place symlinks to newly compiled ustreamer.

Directory /root/code/ustreamer/
```
$ make install
$ ln -sf /usr/local/bin/ustreamer /usr/bin/
$ ln -sf /usr/local/bin/ustreamer-dump /usr/bin/
```

### Configure override.yaml

Append this to the end of /etc/kvmd/override.yaml

```
kvmd:
    streamer:
        forever: true
        cmd_append:
            - "--format-swap-rgb"
            - "--buffers=8"
            - "--format=rgb24"
            - "--encoder=cpu"
            - "--v4p"
```

### Restart kvmd Service

Restart kvmd service

```
systemctl restart kvmd
```

**Result:** _After restarting kvmd service, HDMI passthrough is enabled from HDMI0 port_


### Root Read Only Mode and Exit

```
$ ro
$ exit
```

### Update Ustreamer In The Future

Same process but all you need to do is go to rw mode and then go to the /root/code/ustreamer directory

```
$ rw
$ Password: root
$ cd /root/code/ustreamer
$ git pull
$ make clean
$ make WITH_GPIO=1 WITH_SYSTEMD=1 WITH_JANUS=1 WITH_V4P=1
$ make install
$ systemctl restart kvmd
$ ro
```

## RPi4/DIY/etc

For Pi 4 model B based platforms: append to /etc/kvmd/override.yaml
```
kvmd:
    streamer:
        forever: true
        cmd_append:
            - "--format-swap-rgb"
            - "--buffers=8"
            - "--format=rgb24"
            - "--encoder=cpu"
            - "--v4p"
```

### Restart kvmd Sevice
 
```
systemctl restart kvmd
```

**Result:** _After restarting kvmd service, HDMI passthrough is enabled from HDMI1 port (the 2nd micro HDMI port counting from the USB power port)_

## RPi3

### Baiscally Same As Above

Extra line change on line 83.

Pull ustreamer source from repo

Edit ~/ustreamer/src/libs/drm/drm.c 

line [83](https://github.com/pikvm/ustreamer/blob/c848756d53626d2ba462a698777c6f4e32bf100c/src/libs/drm/drm.c#L83), replace platform-gpu-card with platform-soc:gpu-card

line [84](https://github.com/pikvm/ustreamer/blob/c848756d53626d2ba462a698777c6f4e32bf100c/src/libs/drm/drm.c#L84), replace HDMI-A-2 with HDMI-A-1

Compile ustreamer the same way as above

Install and link compiled ustreamer the same way as above.

Then apply the same changes to /etc/kvmd/override.yaml as described in RPi4 above.

## Reverting Back To Stock

To revert to stock ustreamer, just remove the created symlinks in /usr/bin/

### Delete Customized Ustreamer
```
$ su -
$ Password: root
$ rw
$ rm /usr/bin/ustreamer
$ rm /usr/bin/ustreamer-dump
```

### Reinstall Distribution Version Of Ustreamer
```
pacman -S ustreamer
ro
```

## Troubleshooting

### No Signal
Use the same resolution on the HOST machine that you want on the output monitor.

Old 720P max res monitor set the desktop resolution to 720P as well etc.

### Fixing EDID for 1080P30Hz

Pending...
