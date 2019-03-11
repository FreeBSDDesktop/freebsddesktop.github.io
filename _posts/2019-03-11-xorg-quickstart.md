---
author: wright
---

# Xorg Quickstart

Everyone knows that FreeBSD is a great platform for building appliances and running servers in your datacenter.  One thing that I think FreeBSD doesn't get much credit for is how easy it is to set it up as a laptop or desktop workstation.  This blog post will provide a quickstart for new users or people who infrequently setup desktop systems.  

It will be short on details, but there are plenty of resources if avaialbe in previous [blog posts](https://freebsddesktop.github.io/2018/12/08/drm-kmod-primer.html), or even on the [FreeBSD Wiki](https://wiki.freebsd.org/Graphics/) if you would like to dive a bit deeper.  We'll assume you have a laptop, or desktop, system with an Intel CPU and integrated Intel Graphics (aka Intel HD Graphics) and a FreeBSD install.  In this case we'll be using a system tracking CURRENT, but these directions should also work on 12.0-RELEASE and newer.

## Overview

The overview of the process is:
1) Install the drm-kmod pkg.
2) Install xorg-server, a window manager (XFCE4) and some useful tools.
3) Launch X.

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
Please *carefully* read the helpful pkg message after this step.  It will explain how to do two very important steps:
1) update rc.conf to load the i915kms driver on reboot
2) add yourself to the video group.  

Let's do that now:

```
$ sudo sysrc kld_list+="/boot/modules/i915kms.ko"
kld_list: if_iwm -> if_iwm /boot/modules/i915kms.ko
$ sudo pw groupmod video -M $USER
```

At this point reboot your system and ensure the kernel module loads successfully.  Assuming it reboots successfully, you can verify the driver is installed by executing:

```
$ dmesg | grep drm
```

Lines prefixed with [drm] indicate the i915kms driver has attached to your system as well as showing some useful debugging info.

### Install Xorg and XFCE4

Now that we've verified our DRM driver is loaded let's get to the fun stuff and install Xorg a GUI and some web browsers:

```
$ sudo pkg install xorg-server xfce firefox chromium
```

Once this command completes you'll be good to go.  One nice feature is that the `xfce` package is actually a meta-package that makes sure XFCE gets installed along with many useful utilities.

### Start Xorg

So now we are all set to dive into XFCE!  In my environment I just add a single line to my .xinitrc file like so to ensure XFCE is picked up:

```
$ echo "exec startxfce4" >> ~/.xinitrc
```

Then it is a matter of just running `startx` like so.  XFCE should be launched and away we go:

```
$ startx
```

## Next Steps

I hope this guide is helpful for you all.  You will notice that there is now no need to configure an `xorg.conf` file.  You can still do more complex configurations with custom config files, but for most users things should just work.  Also, if you have a system with an AMD GPU you would follow the same steps above.  If you read the package message after installing `drm-kmod` you will see instructions on how to enable that.

If you run into problems don't hesitate to reach out to us on the freebsd-x11@ mailing list or on Gitter.im where we hang out in the "FreeBSDDesktop" room.

Happy hacking!
