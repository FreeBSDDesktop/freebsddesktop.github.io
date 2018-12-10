---
author: wright
---

# Getting Started With drm-kmod

It is a very exciting time for FreeBSD graphics.  With the 12.0-RELEASE we have put in a lot of effort to provide
support for modern graphics hardware.  What does this mean?  Well, for example, if you purchased a laptop in the past 10 years
getting graphics support has been hit or miss on FreeBSD.  Some people would opt to purchase a system with an Nvidia GPU,
and rely on the propritary driver they provide.  But what about systems with integrated Intel HD or AMD Radeon GPU's
which arguably compose a majority of market share?  There was no great answer here, forcing the user to use
software accellerated graphics, or worse another OS.  Fortunately this is no longer the case with the introduction of the 
`drm-kmod` port!

This blog post should serve as a primer, providing a bit of background and a walk through of getting things working.  For
the impatient, check out the tl;dr section at the end where you can just copypasta the required commands - but you wouldn't
do that without understanding what's happening right? :)

## What is drm-kmod?

DRM stands for "Direct Rendering Manager" and is term taken from the Linux kernel development community refering to
how they solved the issue to dealing with newer graphics processors.  It exposes an API that lets user-space applications
directly interact with GPU's among other thing.  There is a great wikipedia entry [here](https://en.wikipedia.org/wiki/Direct_Rendering_Manager)
on it that should help clarify a lot of the confusing vocabulary.

FreeBSD has approached gaining support for these modern GPUs by levaraging the work already done by the Linux
community.  A kernel module for your given GPU is loaded (one for Intel chips, and another for AMD).  These drivers then
interact with FreeBSD via our updated `lkpi` (Linux Kernel Programming Interface).  This approach has allowed us to import
the Linux drivers with relatively minor modifications which hopefully will ease long-term FreeBSD support.  We'll have future
posts on how the `lkpi` works on FreeBSD, but for now you just need to know that we can now closely follow upstream Linux 
driver development for GPU's for use on FreeBSD.  Infact we often can support the lastest Linux driver code before many Linux
distributions have released their updated packages (I'm writting this post on a system using the Linux 4.18 DRM code as we
speak, whereas my Debian-9 systems are running the 4.9 Linux kernel).  Pretty slick stuff if you ask me.

## So how does it work?

In addition to making long term support easier, lots of effort has gone into making the process of installing these drivers
as seamless as possible.  Hopefully we will have an update for `bsdinstall` to automate this setup process soon, but for now 
it's actually quite easy.  So what do you need?

1. A system with Haswell or newer Intel HD Graphics (see [here](https://en.wikipedia.org/wiki/Intel_Graphics_Technology) for a list)
   or a system with AMD Radeon HD7000 AMD GPU or newer (see [here](https://en.wikipedia.org/wiki/Radeon_HD_7000_Series) for a list).
   
2. A system running FreeBSD 11.2-RELEASE or newer.

### Validate Hardware Is Supported

After installing a fresh copy of 12.0-RELEASE it is easy to determine which generation of graphics processor you have.
When you system has rebooted after installation log in and run pciconf, I find it easier to pipe it through `less`.  

We are going to look for what is registered under `vgapci`:

```
$ pciconf -lv|less
<unrelated output>

vgapci0@pci0:0:2:0:     class=0x030000 card=0x85091558 chip=0x591b8086 rev=0x04 hdr=0x00
    vendor     = 'Intel Corporation'
    device     = 'HD Graphics 630'
    class      = display
    subclass   = VGA
```

Ok there we go, pciconf reports that have a `Intel HD Graphics 630` device (this is a Kaby Lake generation laptop) so I 
should be good to go.  If your system has a AMD Radeon GPU installed you should it registered here as well.  If nothing 
is reported its possible you have an older system, we'll cover what to do in those cases later on.

So, the good news is I have a system whose graphics are being detected as being available so now I can install the 
`drm-kmod`  port.  So let's do it!

### Installing the port

As mentioned earlier a lot of effort has gone into making this process easy for the end-user, hopefully we'll 
illustrate that here.  

To install the driver for your system all you have to do is install the `drm-kmod` port or package.
I prefer to use packages in this case, but compiling from the ports tree works just as well.  Let's see what that
installation looks like:

```
$ pkg search drm-kmod
drm-kmod-g20180930             Metaport of DRM modules for the linuxkpi-based KMS components
$
$ sudo pkg install drm-kmod
Updating FreeBSD repository catalogue...
FreeBSD repository is up to date.
All repositories are up to date.
The following 2 package(s) will be affected (of 0 checked):

New packages to be INSTALLED:
	drm-kmod: g20180930 [FreeBSD]
	drm-next-kmod: 4.11.g20181027_1 [FreeBSD]

Number of packages to be installed: 2

The process will require 8 MiB more space.
1 MiB to be downloaded.

Proceed with this action? [y/N]: y
[1/1] Fetching drm-next-kmod-4.11.g20181027_1.txz: 100%    1 MiB   1.5MB/s    00:01    
Checking integrity... done (0 conflicting)
[1/2] Installing drm-next-kmod-4.11.g20181027_1...
[1/2] Extracting drm-next-kmod-4.11.g20181027_1: 100%
[2/2] Installing drm-kmod-g20180930...
Message from drm-next-kmod-4.11.g20181027_1:

The drm-next-kmod port can be enabled for amdgpu (for AMD GPUs starting with
the HD7000 series / Tahiti) or i915kms (for Intel APUs starting with HD3000 /
Sandy Bridge) through kld_list in /etc/rc.conf. radeonkms for older AMD GPUs
can be loaded and there are some positive reports if EFI boot is NOT enabled
(similar to amdgpu).

For amdgpu: kld_list="amdgpu"
For Intel: kld_list="/boot/modules/i915kms.ko"
For radeonkms: kld_list="/boot/modules/radeonkms.ko"

Please ensure that all users requiring graphics are members of the
"video" group.

Older generations are supported by the legacy kms modules (radeonkms / 
i915kms) in base or by installing graphics/drm-legacy-kmod.
```

At this point it is **very** important to read the post install message.  We wil note the critical details here:

* The `drm-kmod` port is actually a meta-port, it figures out the correct version of the driver itself to install on your
  system.  In my case it has installed the `drm-next-kmod-4.11`.  
  * The `4.11` bit refers to which Linux kernel driver we are using.  This is a great way to also verify if you have a really
    bleeding edge system what are the chances of it being supported.  If there is Linux support for your chip chances are
    pretty good we'll also support it.  If not let [us](https://lists.freebsd.org/mailman/listinfo/freebsd-x11) know! :)

* The post-install message tells you how to enable the driver, it is a matter of updating `rc.conf` with the appropriate
  `kld_list` parameter.  In my case my `kld_list` looks like so:
   `kld_list="/boot/modules/i915kms.ko"`
   
* The final step is most important, make sure your user is a member of the `video` group!  Otherwise when you start Xorg
  you will not have permissions to access the `/dev/drm/` devices, this will result in Xorg failing to start.


So, after following the above post-install steps it is just a matter of restarting your system to ensure the kmod is
loaded cleanly and starting your preferred window manager.  

Seriously that's it - oh, and one final note:

**You do not need to install any of the `xf86-video` drivers or provide an Xorg configuration.  Autodetection should "just work".**

### Validate Hardware Accelleration Is Working

If you want to make sure your GPU is enabled you can look through `dmesg` and search for lines prefaced with `[drm]`, this should
show the kernel module being loaded - in my case the last relevant line says the following:

```
[drm] Initialized i915 1.6.0 20170123 for drmn0 on minor 0
```

Futhermore, you can also install the `mesa-demos` package and search for `renderer` when you run `glxinfo` like so:

```
$ glxinfo |grep renderer
    GLX_MESA_multithread_makecurrent, GLX_MESA_query_renderer, 
    GLX_MESA_query_renderer, GLX_MESA_swap_control, GLX_OML_swap_method, 
Extended renderer info (GLX_MESA_query_renderer):
OpenGL renderer string: Mesa DRI Intel(R) HD Graphics 630 (Kaby Lake GT2) 
```

Huzzah - success!  Now time to play some games, watch some videos or even get some work done :)

## Support For Older Systems
So what if you have a system older than a Haswell era Intel CPU?  Historically the driver for this hardware has been
included in the base FreeBSD OS and will continue to be so in 12.0-RELEASE, but the expectation is it will be removed by 
version 13.  To prepare for this eventuality a new port has been created named `drm-legacy-kmod`.  If you have one of these
older systems and are having problems with graphics using the `drm-kmod` port, please try this legacy version.


## Final Thoughts

Hope this blog post has been helpful.  When I started helping work on this part of FreeBSD it was due to the fact that
I had a laptop from work which I couldn't use for my Sysadmin work due to a lack of supported graphics.  I then noticed
a lot of developers recently have been doing much of their work on Apple laptops, which is pretty understandable - I lug
around two laptops daily since there are still some rough edges I need to use macOS for.  But my goal was to help get things
updated enough so that more developers could dogfood the code they are working on.  Hopefully this will happen, but if
not at least I was able to scratch an itch :)

Also, we've updated the FreeBSD graphics wiki which you should check out:
[https://wiki.freebsd.org/Graphics](https://wiki.freebsd.org/Graphics)

The plan is to keep this more regularly updated as support for more platforms and chipsets are brought onboard.


## Commands & Configs For The Impatient

* Install the `drm-kmod` meta-port:
```
$ sudo pkg install drm-kmod
```

* Ensure kernel module is loaded at boot:
(*Substitute `amdgpu` for `i915kms` if you have a newer AMD GPU*)
```
$ sudo sysrc kld_list+="/boot/modules/i915kms.ko"
kld_list: if_iwm -> if_iwm /boot/modules/i915kms.ko
$
```
  
* Ensure your UID is a member of the "video" group:
```
$ sudo pw groupmod video -M $USER
```

* Reboot system and start X normally

