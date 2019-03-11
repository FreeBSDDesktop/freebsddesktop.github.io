---
author: wright
---

# Xorg Quickstart

One of the great things about using FreeBSD as a graphical workstation is how simple it is to get a fully working environment with just a handful of commands.  This blog post will provide a quickstart for new users, or people who infrequently setup desktop systems.  It will be short on details, but there are plenty of resources if avaialbe in previous [blog posts](https://freebsddesktop.github.io/2018/12/08/drm-kmod-primer.html), or even on the [FreeBSD Wiki](https://wiki.freebsd.org/Graphics/) if you would like to dive a bit deeper.  This walkthrough assumes you have a laptop, or desktop, system with an Intel CPU and integrated Intel Graphics (aka Intel HD Graphics) and a FreeBSD install.  In this case we'll be using a system tracking CURRENT, but these directions should also work on 12.0-RELEASE and newer.

## Overview

THe overview of the process is:
1) Install the drm-kmod pkg
2) Install xorg-server and a window manager (XFCE4)
3) Launch X

## Walkthrough

### Install the drm-kmod pkg (output edited for brevity)
```
$ sudo pkg install drm-kmod
New packages to be INSTALLED:
	drm-kmod: g20181126 [FreeBSD]
	drm-current-kmod: 4.16.g20190305 [FreeBSD]
Message from drm-current-kmod-4.16.g20190305:

The experimental drm-current-kmod port can be enabled for amdgpu (for AMD
GPUs starting with the HD7000 series / Tahiti) or i915kms (for Intel
APUs starting with HD3000 / Sandy Bridge) through kld_list in
/etc/rc.conf. radeonkms for older AMD GPUs can be loaded and there are
some positive reports if EFI boot is NOT enabled (similar to amdgpu).

For amdgpu: kld_list="amdgpu"
For Intel: kld_list="/boot/modules/i915kms.ko"
For radeonkms: kld_list="/boot/modules/radeonkms.ko"

Please ensure that all users requiring graphics are members of the
"video" group.

Older generations are supported by the legacy kms modules (radeonkms / 
i915kms) in base or by installing graphics/drm-legacy-kmod.
```
Please carefully ready the helpful pkg message here, it will intruct you to do two very important steps - update rc.conf to load the i915kms driver on reboot, and add yourself to the video group.  Let's do that now:

```
$ sudo sysrc kld_list+="/boot/modules/i915kms.ko"
kld_list: if_iwm -> if_iwm /boot/modules/i915kms.ko
$ sudo pw groupmod video -M $USER
```

At this point reboot your system, and ensure the kernel module loads successfully.  This can be done by executing:

```
$ dmesg | grep drm
```

### Install Xorg and XFCE4

Now that we've verified our DRM driver is loaded let's get to the fun stuff and install Xorg a GUI and some web browsers:

```
$ sudo pkg install xorg-server xfce firefox chromium
```

The `xfce` pkg is a meta-package and will enusre all dependencies and tools required for a useful desktop environment are installed, we also install firefox and the opensource version of Chrome named Chromium.  If you want to see what other packages are available there is always `pkg search $keyword` where keyword is the name of your tool.

### Start Xorg

So now we are all set to dive into XFCE!  In my environmet I just add a single line to my .xinitrc file like so to ensure XFCE is picked up:

```
$ echo "exec startxfce4" >> ~/.xinitrc
```

Then it is a matter of just running `startx` like so.  XFCE should be launched and away we go:

```
$ startx
```

## Next Steps

I hope this guide is helpful for you all.  You will notice that there is now no need to configure an `xorg.conf` file.  You can still do more complex configurations with custom config files, but for most users things should just work.  If you run into problems don't hesitate to reach out to us on the freebsd-x11@ mailing list or on Gitter.im where we hang out in the "FreeBSDDesktop" room.

Happy hacking!
