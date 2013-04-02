---
layout: post
title: "Imagine: an Automatic Centos 6 base image creation using bash"
category: IT
tags: [DevOps, provisioning]
---
{% include JB/setup %}

Imagine is a small tool that creates a Vagrant base image. I maintain for my personal use and learning.
Other tools like [VeeWee](https://github.com/jedi4ever/veewee) or [BoxGrinder](http://boxgrinder.org/)
would do the job very well but I wanted to get my hands dirty and solve the same problem using good old Bash.

To achieve our goal, we need the official Centos 6 ISO image, a Bash script and a kickstart script.

This is a work in progress:
* the script is geared toward OSX and you will need to adapt it for other operating systems,
* some of the hardcoded code would need to adapted to your requirement.

Please find the code on [github](https://github.com/emeka/imagine) and read
the [README](https://github.com/emeka/imagine/blob/master/README.md) file.

## 1. Command Line

If you look under imagine/virtualbox, you will find the create-base-image command which is used like this:

<blockquote>
    create-base-image vmname iso_file kickstart_file
</blockquote>

where
* *vmname* is the name of the base image,
* *iso_file* is the path to the download Centos 6 ISO on your file system,
* *kickstart_file* is the kickstart file that will be used to customize your base image.

You will need to download Centos 6 base image from a
[mirror](http://isoredirect.centos.org/centos/6/isos/x86_64/) near you.

If you have download the CentOS-6.3-x86_64-bin-DVD1.iso base image in the /tmp directory,
the following command will automatically create a new base image with the vagrant user
in the ~/stores/ovf/test directory.

<blockquote>
./create-base-image test /tmp/CentOS-6.3-x86_64-bin-DVD1.iso centos6-ks.cfg
</blockquote>

You can change the target directory either by modifying the Bash script or by setting the OVF_STORE to the root
your ovf store.

## 2. The Code

The code is separated in two parts:
1. the Bash script that interacts with VirtualBox using the VBoxManage command line,
2. the kickstart script that provides the initial configuration and the post install scripts.

### 2.1 Bash script

The role of the [Bash script](https://github.com/emeka/imagine/blob/master/virtualbox/create-base-image) is to

* create a new virtual machine,
* create an empty hard drive image and mount it,
* create a floppy image countaining the kickstart script and mount it using code from [here][1],
* mount the OS iso image,
* boot the VM,
* select the kickstart boot script from the floppy using the BoxManage controlvm keyboardputscancode command,
* reboot several times to allow the post installation script to run and clean itself.

The main trick is to use a mounted floppy to allow kickstart installation without having to rely on any network
kickstart server and the related DHCP configuration.  Thanks to VirtualBox ability to send keystrokes to a VM,
as described [here][2], we can automatically select the kickstart file on the floppy disk on boot.

### 2.2 Kickstart script

The [kickstart script](https://github.com/emeka/imagine/blob/master/virtualbox/centos6-ks.cfg) is a template.
Any ${variable name} place holder will be replaced by the corresponding exported variable from the bash
script using a simple Perl command.

In addition to the standard kickstart commands, the script will:
* create a local user with name ${USERNAME},
* copy your public ssh key in this user authorized_keys file,
* create a post installation script under /etc/rc3.d.

The post installation script will be run during the next reboot and, in this case, will install VirtualBox Guest
Additions. It will then clean itself.


## Conclusion

This little tool shows how simple it is to create and customize a base image using simple tools like Bash
and kickstart. Hopefully, you will find it usefull.



[1]: http://www.jedi.be/blog/2009/11/17/commandline-creation-of-msdos-floppy-on-macosx/
[2]: http://www.jedi.be/blog/2010/08/29/sending-keystrokes-to-your-virtual-machines-using-X-vnc-rdp-or-native/