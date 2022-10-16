# ubuntu-22-nvidia-suspend-fix-script
Fix for suspend not working on some instances of Ubuntu 22 w/Nvidia drivers

# Ubuntu 22 LTS w/Nvidia Drivers, sometimes causes courrpted text and or issues with suspend.

Issue referenced here:

- https://bugs.launchpad.net/ubuntu/+source/mutter/+bug/1876632
- https://gitlab.gnome.org/GNOME/mutter/-/issues/1942


_Amazing_ solution, found on Nvidia forums, created by **devyn.cairns**

https://forums.developer.nvidia.com/t/trouble-suspending-with-510-39-01-linux-5-16-0-freezing-of-tasks-failed-after-20-009-seconds/200933/12

```
gnome-shell is trying to talk to the NVIDIA driver after it has already gone into suspend, so it can’t respond. Linux tries to freeze the task, but fails because gnome-shell is waiting for a response from the driver and can’t be frozen.

The solution is to manually suspend gnome-shell using the STOP signal before the NVIDIA driver goes to suspend. Then use the CONT signal on resume.
```

# The following steps helped solve the issue for me

### Create the following files (and chmod them)


```
sudo vim /usr/local/bin/suspend-gnome-shell.sh
chmod +x /usr/local/bin/suspend-gnome-shell.sh
```


```bash
#!/bin/bash

case "$1" in
    suspend)
        killall -STOP gnome-shell
        ;;
    resume)
        killall -CONT gnome-shell
        ;;
esac
```

```
/etc/systemd/system/gnome-shell-suspend.service
```

```bash
[Unit]
Description=Suspend gnome-shell
Before=systemd-suspend.service
Before=systemd-hibernate.service
Before=nvidia-suspend.service
Before=nvidia-hibernate.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/suspend-gnome-shell.sh suspend

[Install]
WantedBy=systemd-suspend.service
WantedBy=systemd-hibernate.service
```

```
/etc/systemd/system/gnome-shell-resume.service
```

```bash
[Unit]
Description=Resume gnome-shell
After=systemd-suspend.service
After=systemd-hibernate.service
After=nvidia-resume.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/suspend-gnome-shell.sh resume

[Install]
WantedBy=systemd-suspend.service
WantedBy=systemd-hibernate.service
```

### Finally, run

```
systemctl daemon-reload
systemctl enable gnome-shell-suspend
systemctl enable gnome-shell-resume
```

### After doing this, I was able to successfully suspend on Ubuntu 22 on Wayland.
