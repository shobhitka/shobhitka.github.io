---
layout: post
title:  "Create Customized Linux Distro"
date:   2012-07-16 15:46:17 +0530
categories: Linux
---
I own a HP Mini210 with Intel Pineview processor and all of the latest and greatest Linux distro sucks on this one. And those Win guys who may be reading this, don't ever mention to me on trying Win 7 on this one, though it came with home basic by default, that goes against my morals to have such a thing like Windows on my machine less even talk about it.

So here I am on creating my own distribution customized to my taste and perfect the way I want it.

Few questions I asked my self about what I want to do once I am done with this distribution and thats what I concluded - 

1. Boot as fast as it can
2. Able to browse internet
3. Download my mails
4. Watch on-line/local stored movies. HD content possible ?

So the above needed only simple packages, but I wanted something more - An easy to use package management system and I zeroed on Debian tool "aptitude" and my base distro as Debian. So lets jump into creating it right away.

**What you need to achieve this**

* Any Linux desktop environment - Fedora/Ubuntu/Mint/Suse/Mandrake, etc
* I see pain on Windows guys :) So install
  * VirtualBox
  * And then install any of the above mentioned flavors of Linux distros inside that
* Do install debootstrap, make, gcc

**Creating the base root file system**

Now you can directly install this on a physical hard disk as well, but I will give steps based on virtual disks so that we can burn them as Live CD, you know the cool stuff to boot from USB pen-drive any-time, anywhere and yes persistent.

Create a virtual disk -

{% highlight shell %}
dd if=/dev/zero of=rootfs.ext4 bs=1024 count=524288
{% endhighlight %}

Yeah 512 MB is more than sufficient. We can resize it later. Keeping it small helps in actually while burning on USB pen drive. Actually the base console install needs just 256 MB. Blown Away Eh :)

Prepare the disk image file with ext4 file system and mount it. Next create the basic root fs.

{% highlight shell %}
mkfs.ext4 rootfs.ext4
mkdir rootfs
mount -o loop rootfs.ext4 rootfs
debootstrap --arch ia64 squeeze rootfs http://ftp.us.debian.org/debian
{% endhighlight %}

You have the rootfs ready and now time to use another cool thing - chroot

{% highlight shell %}
mount -t proc proc rootfs/proc
mount -t sysfs sys rootfs/sys
mount --bind /dev rootfs/dev
chroot rootfs /bin/bash
{% endhighlight %}

You are now in your root file system and any modification you do are restricted to your new debian root fs. Do configure the networking stuff, like modifying the /etc/hosts, /etc/network/interfaces, /etc/resolv.conf, /etc/hostname, etc

Also remember to add a new user for your self using useradd and also change the root password using passwd command. In Fedora selinux might crip, so disable that. Now our rootfs is ready. Just unmount everything

{% highlight shell %}
exit
umount rootfs/proc
umount rootfs/sys
umount rootfs/dev
umount rootfs
{% endhighlight %}

**Building the Kernel**

I wanted to use the latest Linux kernel out there and grabbed the tarball from www.kernel.org for latest stable 3.4.4 kernel

Apply the default kernel configuration **x86_64_defconfig**
Now here comes the difficult and iterative task of fine tuning your kernel to support your device and only your device :) I removed all the unnecessary drivers. Just selected exact drivers matching my hardware for the HP Mini 210. Sometimes things didn't work and you relise you have removed wrong driver and need to rebring it in. Now as I wanted to make it boot fast. Most of the drivers for the netbook I just made as kernel modules to have a very small kernel. Actually basic kernel booted in less than 3 seconds. 

**Putting things together with a boot-loader (Grub)**

Insert you pen drive and make two partitions using fdisk. Note the device name carefully - sdb or sdc, etc

1. boot partition - 64 MB is more than sufficient. Format as vfat(mkfs.vfat)
2. rootfs partition - 8 GB for now. You can resize it later
3. Make additional partitions as you wish. Basically the current system to be usable you may need nothing more than 1.5 GB

Mount the vfat partition say as /media/boot. Install the grub as follows -
{% highlight shell %}
grub2-install --force --no-floppy --root-directory=/media/boot /dev/sdb1
{% endhighlight %}

Copy the default grub.cfg from your host linux system /boot/grub2/grub.cfg /mnt/boot/boot/grub2/grub.cfg. Remove all the enteries and just add a new entry for our latest kernel as follows -

{% highlight shell %}
menuentry 'TUX Kernel (3.4.4)' --class gnu-linux --class gnu --class os {
	load_video
	set gfxpayload=keep
	insmod gzio
	insmod part_msdos
	insmod ext2
	set root='(hd0,msdos1)'
	echo 'Loading Tux Kernel (3.4.4)'
	linux   /bzImage root=/dev/sdb2
}
{% endhighlight %}

Burn the rootfs, resize it to 8GB and label it

{% highlight shell %}
dd if=rootfs.ext4 of=/dev/sdb2
e2fsck /dev/sdb2
resize2fs /dev/sdb2
e2label /dev/sdb2 rootfs
{% endhighlight %}

Copy the kernel image to the /boot partition

{% highlight shell %}
cp arch/x86/boot/bzImage /media/boot
{% endhighlight %}

We are ready. just boot from the pen drive and you have console environment booted in under 8s. Install the X environment as  follows -

{% highlight shell %}
apt-get update
apt-get install xfce4
{% endhighlight %}

Now we have a bare minimum Linux system (Under 1 GB). We can fine tune what packages we want like firefox(browser), thunderbird (e-mail client), vlc(media player) etc avoiding all the clutter that all of the latest distros install by default. I got my desired system in less than 1.5Gb booting in around 10s.
