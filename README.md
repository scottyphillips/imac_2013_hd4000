# Enabling HD4000 Graphics on iMac 2013 with broken GTX780M for use in Linux. 

My GTX780M Kepler in my 2013 iMac decided that it was time to die. One moment it was working and then it crashed and I needed to hard reboot. When I rebooted, Linux refused to initiate the card. I suspect that either GPU RAM or something firmware related has gone bad. But when booting, parts of the graphics subsystem still appeared to be working. 

It is fairly well known that many Intel Macbooks have a built in iGPU that is typically hidden from users and requires advanced tweaking typically at the EFI level to expose it to operating systems. Turns out that my iMac has a hidden Intel HD4000 as well. This is the basic steps to enable it in the event your Nvidia dGPU kicks the bucket. 

These are quick notes I will write something up later. 

# Requirements
* Working REFIND bootloader
* Know how to use REFIND
* Linux needs to be booted from REFIND in EFI mode, not CSM. (I use Arch btw)
* Some way to access the linux Terminal (eg SSH)

# REFIND config

This is the working config for 'refind.conf'

```
# Launch specified OSes in graphics mode. By default, rEFInd switches
# to text mode and displays basic pre-launch information when launching
# all OSes except macOS. Using graphics mode can produce a more seamless
# transition, but displays no information, which can make matters
# difficult if you must debug a problem. Also, on at least one known
# computer, using graphics mode prevents a crash when using the Linux
# kernel's EFI stub loader. You can specify an empty list to boot all
# OSes in text mode.
# Valid options:
#   osx     - macOS
#   linux   - A Linux kernel with EFI stub loader
#   elilo   - The ELILO boot loader
#   grub    - The GRUB (Legacy or 2) boot loader
#   windows - Microsoft Windows
# Default value: osx
#
use_graphics_for linux

# Tell a Mac's EFI that macOS is about to be launched, even when it's not.
# This option causes some Macs to initialize their hardware differently than
# when a third-party OS is launched normally. In some cases (particularly on
# Macs with multiple video cards), using this option can cause hardware to
# work that would not otherwise work. On the other hand, using this option
# when it is not necessary can cause hardware (such as keyboards and mice) to
# become inaccessible. Therefore, you should not enable this option if your
# non-Apple OSes work correctly; enable it only if you have problems with
# some hardware devices. When needed, a value of "10.9" usually works, but
# you can experiment with other values. This feature has no effect on
# non-Apple computers.
# The default is inactive (no macOS spoofing is done).
#
spoof_osx_version 10.9

# wiki page on rEFInd (https://wiki.archlinux.org/index.php/rEFInd).
menuentry "EndeavourOS Linux" {
    icon     /EFI/refind/icons/os_arch.png
    set gfxpayload=keep
    outb 0x728 1
    outb 0x710 2
    outb 0x740 2
    outb 0x750 0
    volume   <PARTUUID> # get PARTUUID by running 'sudo blkid'
    ostype   linux
    loader   /boot/vmlinuz-linux
    initrd   /boot/initramfs-linux.img
    options  "root=UUID=<UUID> rw nowatchdog i915.lvds_channel_mode=2 i915.modeset=1 nvme_load=YES resume=UUID=<UUID> loglevel=3"
    submenuentry "Boot using fallback initramfs" {
        initrd /boot/initramfs-linux-fallback.img
    }
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
    # disabled
}

```
Make sure that nouveau is blacklisted and/or Nvidia propriatery graphics are uninstalled.
Make sure 'mesa' and 'xf86-video-intel' is installed. 


# Caveats
* Against the advice of the Arch wiki I had to install 'xf86-video-intel' to get this to work. 
* X11 refused to run hardware accelleration, but strangely Wayland works fine.
* No Screen blanking -> screen is always on. Thats a pain for me because I like to run services in the background. 
* The display is 'noisy' its almost like its using an analog signal to the panel. The artifacts look similar to clock lines where the color signal changes. But with the right combination of colors you can minimise the effect. Black and white colors work best.
* Forget about gaming. Minecraft dropped from locked 120fps on the Nvidia to barely hitting 30fps on the HD4000.
* Appears to be impossible to change resolution from native 2560x1440x60.
  
# Benefits?
* Err, Wayland kinda actually works? (About as well as you expect Wayland to run on a HD4000).
* No more DKMS drivers breaking with every kernel update. 
* Less gaming means more quality time with family
* You can run Linux in pure EFI mode and get access to the TTYs. With Propriatery Nvidia I previously had to run in CSM mode with GRUB.
* EFI mode means faster boot time from REFIND. 


