+++
title = 'Encrypted Gentoo'
date = 2025-08-07T10:46:08+02:00
draft = false
tags = ["guide", "linux"]
+++

 This is a small guide on how to do encrypted Gentoo install. This isn't really an all purpose article for all cases, I just wrote this while I was installing Gentoo on my Thinkpad T430 laptop and wanted to make a article that maybe could help someone a bit.


## Preparing disks
 First, you will have to prepare your disk for the Gentoo installation. This will be the base for your entire system. You will have to create three partitions (boot, swap and root) using the program gdisk and then use the cryptsetup command on the root partition to create encrypted luks container.


### Creating encrypted container

```sh
cryptsetup -yv luksFormat /dev/sda4
```

### Opening the encrypted container

Note that the pharse at the end of the command is the mapper name, you can choose any.

```sh
cryptsetup open /dev/sdb4 root
```
After opening the encrypted container, you can notice that it is located in the `/dev/mapper/root`.


### Formatting the partitions
Now format the boot partition to ext4 and root to btrfs. And don't forget to format swap partition.

```sh 
mkfs.ext4 /dev/sda1
```

```sh 
mkfs.btrfs /dev/sda3
```

```sh 
mkswap /dev/sda2
```
## Preparing the system
 In this step you will need to download the stage3 tarball from the Gentoo website and extract it installation /mnt/gentoo/. In my case <u>I will be using amd64 desktop profile with OpenRC.</u>


### Mounting the disk
First, you will need to create the `/mnt/gentoo` and `/mnt/gentoo/boot` directories.

```sh 
mkdir -p /mnt/gentoo/boot
```

and then mount the partitions.

```sh 
mount /dev/mapper/root /mnt/gentoo
```
```sh 
mount /dev/sda2 /mnt/gentoo/boot
```


### Downloading and extracting the Stage3 tarball
Go to the [Gentoo download website](https://www.gentoo.org/downloads/) and copy the download link of the selected tarball and download it into `/mnt/gentoo/`. 

```sh 
cd /mnt/gentoo
```
```sh 
wget [tarball link]
```

After downloading the tarball, extract it using this command:
```sh 
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```
After it is done extracting the tarball, you can delete the .tar.xz file.


## Configuring compile options
This is very important thing to set, most of your packages will be compiled with the settings you set in the `/mnt/gentoo/etc/portage/make.conf`. I will show you some important things to set for this guide. Use this [wiki page](https://wiki.gentoo.org/wiki//etc/portage/make.conf) for your invididual configuration as an help guide.After

Most importantly don't forget to set your `-march=` to your CPU architecture. If you are compiling on the computer that will the Gentoo used on then just set `-march=native` and if not then go to the [safe CFLAGS](https://wiki.gentoo.org/wiki/Safe_CFLAGS) wiki page and find the code for the CPU that you will use this Gentoo installation on. In my case I will be setting `-march=ivybridge`.

 You should also set your `CPU_FLAGS_X86` to your CPU flags. You can do it by booting into Gentoo liveiso from an usb flash drive on the computer you will use this Gentoo installation on and running the `cpuid2cpuflags` command to list all the tags.

Rust software is spreading like so caled "COVID-19" few years ago, so I would advise you to include `RUSTFLAGS="${COMMON_FLAGS}"` in your make.conf.

You should also include this quality of life option that will automatically append these flags when you will go compile something. `EMERGE_DEFAULT_OPTS="--ask --verbose"`.

 You will want to have faster downloads from the Gentoo mirrors. So you should set the `GENTOO_MIRRORS` to the closest mirror there is to you. You can find mirrors by going to the [Gentoo mirrors page](https://www.gentoo.org/downloads/mirrors/). 

 You also will want to set the `VIDEO_CARD` option so your video drivers get compiled. I will use the `VIDEO_CARDS="intel"` on my ThinkPad because it has an integeated GPU. If you don't know which flag to set, go to the [video cards section](https://wiki.gentoo.org/wiki//etc/portage/make.conf#VIDEO_CARDS) on the make.conf wiki page to find the GPU flag you need. 

 For the Grub bootloader you will want to set `GRUB_PLATFORMS="pc"` since this guide is focused on installation with legacy boot.

 If you compile on a better CPU than the installation will be used on, you can set the `MAKE_OPTS="-j#"` to the availabile resources you have. I am compiling this installation on AMD Ryzen 7 so I will be using `MAKEOPTS="-j16"`. Make sure you will adjust the -j number to the CPU your installation will be used on after you are done with installing Gentoo.

 This is optional, but if you want to use only Free Software, you could opt to set `ACCEPT_LICENSE="-* @FREE"`. I personally set this on all my machines. 

### Example of my make.conf:
```sh
COMMON_FLAGS="-O2 -march=ivybridge -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
RUSTFLAGS="${COMMON_FLAGS}"
CPU_FLAGS_X86="aes mmx mmxext pclmul popcnt sse sse2 sse3 sse3 sse4_1 sse4_2 ssse3"

EMERGE_DEFAULT_OPTS="--ask --verbose"
GENTOO_MIRRORS="https://mirror.wheel.sk/gentoo"
VIDEO_CARDS="intel"
GRUB_PLATFORMS="pc"

MAKEOPTS="-j16"
ACCEPT_LICENSE="-* @FREE"
USE="device-mapper crypt elogind eme-free bluetooth wifi X wayland pipewire sound-server pulseaudio screencast cdda dvd dvdr -ppp -systemd -telemetry -gnome"

LC_MESSAGES=C.utf8
```

## Installing the system
Before chrooting into your system, you should copy the DNS info to your new installation so you can connect to internet.

```sh 
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

Then create a temporary chrooting script `chroot.sh` which will contain these lines:

```sh
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run

# Uncomment three lines below if you are installing from non-gentoo installation media.
#test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
#mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
#chmod 1777 /dev/shm /run/shm


chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

And run the script:
```sh
sh /mnt/gentoo/chroot.sh
```

After running this script, you should be chrooted in your new gentoo installation enviroment.


### Selecting profile
 A profile is a set of specific use flags, variables and version ranges. You need to choose a profile that will suit your use case. This installation is intended for desktop use, so I will select the `default/linux/amd64/23.0/desktop` profile. To view profiles, you need to run this command:

 ```sh 
 eselect profile list
 ```

 Output example: (my output)

 ```sh
...
  [21]  default/linux/amd64/23.0 (stable)
  [22]  default/linux/amd64/23.0/systemd (stable)
  [23]  default/linux/amd64/23.0/desktop (stable) *
  [24]  default/linux/amd64/23.0/desktop/systemd (stable)
  [25]  default/linux/amd64/23.0/desktop/gnome (stable)
...
```

It is most likely that the desktop profile is already selected if you have downloaded the desktop stage3 tarball. If not, then run the command below with the option number at the end. Then the blue asterisk should move to your selected option. For example, in my case I selected the number 23.

```sh
eselect profile set (your number)
```


### Updating the packages
After doing changes to the use flags in make,conf, we will want to update our current packages so they are built in the way we wanted to.

```sh
emerge-webrsync
```

```sh
emerge --ask --verbose --update --deep --newuse @world
```

After the compilation is done, prune useless packages.

```sh
emerge --ask --depclean
```


### Setting the timezone
If you want your system time to be correct, you should set your system's timezone. You can look at all the timezones availabile by running this command:

```sh
ls -l /usr/share/zoneinfo
```

You can also ls deeper into the directory of your continent to see the availabile timezones.

```sh
ls -l /usr/share/zoneinfo/Europe
```

And then echo the timezone into the timezone file. For example, I will use `Europe/Bratislava`.

```sh
echo "Europe/Bratislava" > /etc/timezone
```

And finally, finish it by emerging timezone-data.

```sh
emerge --config sys-libs/timezone-data
```


### Installing better text editor
This is optional, but it is just much better fore editing files.

```sh
emerge vim
```

### Generating locale
Now you will want to generate the locale by editing locale.gen. Just simply uncomment a line with your locale.

```shm
vim /etc/locale.gen
```

If you don't know how to type in locale code of your language, run this command to view supported locales:

```sh
less /usr/share/i18n/SUPPORTED
```

And now just generate the locale.

```sh
locale-gen
```

### Selecting locale
Now list your locales with eselect. It is likely that your locale is already selected.

```sh
eselect locale list
```

and set your locale

```sh 
eselect locale set (number of your option)
```

And now reload your enviroment using following command:

```sh 
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```


## Preparing linux firmware
 Now it is time to install the core of the operating system, the kernel. It will have to be installed bit differently than it is being done normally. You will have to use the `genkernel` command with few use flags.
You might want to use the `linux-firmware` package if you use proprietary drivers for wifi, bluetooth, etc.. But even if you don't need it like me since I have atheros wifi card, you will need to set  `-firmware` in the `/etc/portage/package.use/sys-kernel` file.


### I need proprietary firmware
If you use proprietary drivers, you will need the firmware. You need to add `~amd64` keyword to the `package.accept_keywords` file. 

```sh 
echo "sys-kernel/linux-firmware ~amd64" >> /etc/portage/package.accept_keywords/sys-kernel
```

<u>Only with @FREE ACCEPT_LICENSE option:</u> Adding license of the package to portage config. 
```sh 
echo "linux-fw-redistributable license(s)" >> /etc/portage/package.license/package.license
```

And then you just install the package.

```sh
emerge sys-kernel/linux-firmware
```



### I don't need proprietary firmware
If you are based and you use devices with free drivers, then you will need to put `-firmware` flag to the package.use. 

```sh 
echo "sys-kernel/genkernel -firmware" >> /etc/portage/package.use/sys-kernel
```


## Installing the kernel
To install kernel, you will need to download two packages. The `genkernel` which is a tool that automates building the kernel. And `gentoo-sources` which includes the kernel source.

```sh 
emerge genkernel gentoo-sources
```

Before you can compile the kernel, you need to select it with the `eselect` command. Do this command to list the availabile kernels:

```sh 
eselect kernel list
```

And then select the kernel. In most cases it will be 1 in new installation.

```sh 
eselect kernel set 1
```

After selecting the kernel, run this command that will start the compiltion of the kernel. The command contains necessary flags for encrypted installation.
**Note:** This might take long time

```sh 
genkernel --luks --lvm --busybox all
```

## Generating Fstab
All of your partitions that will be used in your system must be in the `/etc/fstab` file where are all the mount points and options on how they should be mounted set.

Run the command `blkid` to find the UUIDs of your partitions.

Output example (my output):
```sh
/dev/sda3: UUID="52121e1c-c51c-4ffc-92bc-eb6423e736c8" TYPE="crypto_LUKS" PARTLABEL="Linux filesystem" PARTUUID="389956c6-89c7-4122-b45f-17e6256be3c9"
/dev/sda1: UUID="8bee939e-50f2-41ba-b359-2070ec1511f1" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="eccfcd37-5a98-4e4b-ac18-f1194a9679b9"
/dev/sda2: UUID="75fca4da-eb35-47f7-8696-ae9bbcf1c75c" TYPE="swap" PARTLABEL="Linux swap" PARTUUID="c99f141c-3cea-440c-beb6-cf7fa9c74a89"
/dev/mapper/root: UUID="1e0b2067-8937-47c0-9a5c-f200fe4607b8" UUID_SUB="1ff8b4f2-f1bb-4916-aea9-bdb7f0111584" BLOCK_SIZE="4096" TYPE="btrfs"
```

You will want to put your LUKS parition as root partition in the fstab, in my case it is `/dev/sdb3`.

Fill in the fstab file so it looks like the example below, just input your own UUIDs instead of the one shown in example.

```sh
# /dev/sda3 (root)
UUID=52121e1c-c51c-4ffc-92bc-eb6423e736c8       /       btrfs   rw,relatime    0 0

# /dev/sda1 (boot)
UUID=8bee939e-50f2-41ba-b359-2070ec1511f1       /boot   ext4    rw,relatime     0 2

# /dev/sda2 (swap)
UUID=75fca4da-eb35-47f7-8696-ae9bbcf1c75c       none    swap    defaults        0 0
```

## Setting a hostname
Setting a name of your device which is viewable on your local network.

```sh
echo ClawPad > /etc/hostname 
```

## Enabling DHCPd
If you want to use internet on your device, you will need to install this dhcp server and set startup of the service on the system boot.

```sh
emerge net-misc/dhcpcd
```

```sh
rc-update add dhcpcd default
```

If you want to use wifi on your computer, you should also install this package.

```sh 
emerge --ask net-wireless/iw net-wireless/wpa_supplicant
```



## Setting the root password
To set your root password, use the `passwd` command.

## Installing useful tools
You can skip this step, but it is pretty useful to install some of these packages. Especially these progs packages 

```sh 
emerge sys-fs/btrfs-progs sys-fs/e2fsprogs net-misc/chrony app-shells/bash-completion sys-apps/mlocate sys-process/dcron
```

Now enable the chrony service if you have installed it.

```sh 
rc-update add chronyd default
```

## Installing the bootloader
After succesfuly finishing all the previous steps, you shall proceed to the installation of grub. Grub is a software that helps you boot your system upon pressing the power button on your pc.

Install the grub package and make sure you have `GRUB_PLATFORMS="pc"` in your emerge configs.

```sh 
emerge --ask --verbose sys-boot/grub
```

## Configuring the bootloader
Now you will need to configure your grub to work with encryption. You can find the configuration file at `/etc/default/grub/grub`.

Your grub config should look something like this, change only the shown parameters. Make sure to put your LUKS partition instead of the one in my example. It is the same UUID as in fstab root parition.

```sh
...
GRUB_PRELOAD_MODULES='part_gpt part_msdos luks lvm'
GRUB_CMDLINE_LINUX='dolvm crypt_root=UUID=52121e1c-c51c-4ffc-92bc-eb6423e736c8 root_trim=yes rootfstype=btrfs'
GRUB_ENABLE_CRYPTODISK=y
...
```
Install grub. Use the force flag too so suppress warnings about missing bios boot partition.

```sh
grub-install -f /dev/sda
```


Now generate grub config.

```sh 
grub-mkconfig -o /boot/grub/grub.cfg
```
## Creating an user account
You will probably want to create an user account. Refer to the Gentoo's [user creation section](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Finalizing) in their installation handbook for more information.

Create your account with basic groups:
```sh 
useradd -m -G users,wheel,audio,wheel -s /bin/bash waffelo
```

Now set the password of your user:
```sh
passwd waffelo
```

You will most likely want to use sudo. You will have to add your user to the sudoers file. You will need to go to the `/etc/sudoers` file and uncomment one line so it looks like this:
```sh 
# Allow members of group sudo to execute any command
%wheel   ALL=(ALL:ALL) ALL
```

You will have to save and exit from the file using `:wq!`.


## Leaving chroot
Now that you are done, you will need to exit your installation enviroment safely.

```sh
exit
```

```sh
cd
```

```sh
umount -l /mnt/gentoo/dev{/shm,/pts,}
```

```sh
umount -R /mnt/gentoo
```

Now you can reboot into your system..




