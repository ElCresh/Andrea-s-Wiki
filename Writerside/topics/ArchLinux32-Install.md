# ArchLinux32 - Install guide

![ArchLinux Logo](arch-logo.png){ width=290 }{border-effect=line}

In this topic we are defining a basic white-paper of guidance for ArchLinux32 install

## Downloading
As first things we need to download the latest ISO. This can be done on the following URL: 
<a href="https://mirror.archlinux32.org/archisos/">ArchLinux32 - Mirror</a>

Warning!
: ArchLinux32 ISO are not built frequently so the last available one could be almost 1 year old. Keep this in mind for 
the following steps.

## Preliminary check and configuration
After booting into the iso some checks are needed to ensure that we have a basic internet connection and also a
recognised drive to use for the installation. We also need to configure keymap if we don't use a _US Keyboard_.

### Configure keymap

We can see all availale keympas with the following command:

```shell
ls /usr/share/kbd/keymaps/**/*.map.gz
```

To set keyboard layout we nee to use _loadkeys_ command. For example:

```shell
loadkeys de-latin1
```

### Verify boot mode
Unless you are using some Intel Atom device that are only able on 32-bit UEFI you most probably are booting in 
Legacy mode. To be sure of your current boot mode you can use the following command:

```shell
cat /sys/firmware/efi/fw_platform_size
```

If this command return:
- **64**: the system is booted in UEFI mode on 64-bit systems (most unlikely situation on ArchLinux32)
- **32**: the system is booted in UEFI mode on 32-bit systems
- **'empty'**: the system is booted in legacy BIOS mode

### Check internet connection
By default, the live system tries to configure DHCP client on all available connection. If you are using a WWAN adapter
plaase check that is not blocked by _rfkill_.

You can check all enabled interfaces with:
```shell
ip link
```

You can also check all addresses of the interfaces with:
```shell
ip addr
```

As last step you can also check internet connectivity with _ping_:
```shell
ping archlinux.org

```

### Check available disks
You can se all available drives by using the following command:
```shell
fdisk -l
```

Make notes of the drive path that you want o use. Should be a path like `/dev/sd*`,`/dev/nvme*n*` or `/dev/mmcblk*` where * need to be replaced with the
selected one.

## Preparing the drive

### Partitioning

Please take a look to the relative part of the [Arch Wiki](https://wiki.archlinux.org/title/Partitioning).

### Format the partitions
Once the partition schema is chosen and created we can proceed by formatting all the needed partition. For data partition
we can use _Ext_ (suggested _Ext4_). If EFI boot partition is required need to be formatted as _FAT32_.

For example to create a _Ext4_ partition we can use:
```shell
mkfs.ext4 /dev/<device_partition>
```

If you created a swap partition you can initialize it with `mkswap` as follows:
```shell
mkswap /dev/<swap_partition>
```

If you need an EFI system partition you can format it as follows:
```shell
mkfs.fat -F 32 /dev/<efi_system_partition>
```

### Update keyring
Due to usually old iso we need to update all keyring before continue. If we have on the system expired keys `pacman` 
and `pacstrap` can lead to errors during package installation leaving with a partial or unsuccessful installation.

Before updating the keyring we update the repos db:
```shell
pacman -Syy
```

To update keyring we can use the following command:
```shell
pacman -S archlinux-keyring archlinux32-keyring
```

### Mounting partitions

As first things we need to mount our root partition on the /mnt path. We can achieve this with the command:
```shell
mount /dev/<root_partition> /mnt
```

If you need to mount _EFI boot partition_ you need to create /boot directory inside the path we mounted with the
last command and mount it with:
```shell
mount --mkdir /dev/<efi_system_partition> /mnt/boot
```

If you created a _swap_ partition now can be enabled with:
```shell
swapon /dev/<swap_partition>
```

## Installation

### Select the mirrors
Packages to be installed need to be downloaded form a repository on a mirror servers. They are defined in 
`/etc/pacman.d/mirrorlist`. We used them previously for updating keyring. If you want to change them like prefer a 
server next to your location you can do it right now.

This changes will be later copied to the destination system by `pacstrap`.

Warning!
: No software or configuration (except for `/etc/pacman.d/mirrorlist`) get carried over from the live environment to 
the installed system.

### Installing essential packages

Now we are going to use [pacstrap](https://man.archlinux.org/man/pacstrap.8) to install _base_ packages and Linux _kernel_ and _firmware_ for common hardware 
configuration:
```shell
pacstrap -K /mnt base linux linux-firmware
```

## Configuring

### Fstab
At this point the first step into the configuration of the new installation is to define how disk partitions and 
various other block devices are mounted into the final system. To do this we need to create the _fstab_ file located 
in `/etc/fstab`.

We can do this in two ways. The first one by using [fstab](https://man.archlinux.org/man/fstab.5) command that see current mounted folders and automatically
generate fstab file using _UDID_ or _Labels_. The second one is by creating the file by hand (suggested only for
advanced configurations).

<tabs>
    <tab title="Automatic generation">
        To automatically generate fstab you can use fstab command followed by -U (UDID) or -L (Labels):
        <code-block lang="plain text">fstab -U</code-block>
    </tab>
    <tab title="Manual generation">
        If you decide to create manually the fstab file you can follow this example below:
        <code-block lang="xml">
            #  device                                 dir   type   options   dump   fsck
            UUID=0a3407de-014b-458b-b5c1-848e92a327a3 /     ext4   defaults  0      1
            UUID=f9fe0b69-a280-415d-a03a-a32752370dee none  swap   defaults  0      0
            UUID=b411dc99-f0a0-4c87-9e05-184977be8539 /home ext4   defaults  0      2
        </code-block>
    </tab>
</tabs>

### Access the new system root
Now we can change root from the installation media to the new system. To do this we use the arch-chroot as follows:
```shell
arch-chroot /mnt
```

### Configuring Timezone
Configuring the timezone is an easy process. Just a link to the appropriate configuration to `/etc/localtime` is needed.
This can be done with the following command:
```shell
ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime
```

After this we need to run [hwclock](https://man.archlinux.org/man/hwclock.8) to generate `/etc/adjtime` file:
```shell
hwclock --systohc
```

### Localization
At this point we need to configure system and console localization. Currently, we are using en_US.UTF-8 from the
installation media, but we can change our target installation one. First things we need to change `locale-gen`
by editing `/etc/locale-gen` and removing _#_ from desired locale (default is _en_US.UTF-8_).

After this changes we need to rebuild locales by executing:
```shell
locale-gen
```

Now we need to set _LANG_ variable in `/etc/locale.conf`. You can set it by following this example and replacing the 
locale with the one chosen:
```shell
LANG=en_US.UTF-8
```

As last step we need to configure _KEYMAP_ variable in `/etc/vconsole.conf`. This will make persistant any keyboard
layout change that you have done at the beginning of the installation. You can see 
[vconsole.conf](https://man.archlinux.org/man/vconsole.conf.5) for more info. This is an example configuration:
```shell
KEYMAP=de-latin1
```

### Network configuration
The first step is to configure _hostname_ buy editing `/etc/hostname` and setting it with the chosen one. After
this step is suggested to configure a network manager. This is a list of the most popular one to choose. You can
also see more details on the appropriate [ArchWiki page](https://wiki.archlinux.org/title/Network_configuration).

| Software                                                              | CLI                                                      | TUI                                                    | GUI                                                                          | Notes                                                                                                                                                                                           |
|-----------------------------------------------------------------------|----------------------------------------------------------|--------------------------------------------------------|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [dhclient](https://archlinux.org/packages/extra/x86_64/dhclient/)     | No                                                       | No                                                     | No                                                                           | Only support Ethernet connection                                                                                                                                                                |
| [dhcpcd](https://wiki.archlinux.org/title/Dhcpcd)                     | No                                                       | No                                                     | [dhcpcd-ui](https://aur.archlinux.org/packages/dhcpcd-ui)                    | Only support Ethernet connection, for wireless can launch _wap_supplicant_                                                                                                                      |
| [ConnMan](https://wiki.archlinux.org/title/ConnMan)                   | Yes                                                      | Yes                                                    | Yes                                                                          | No PPPoE support, Mobile broadband support via [ofono](https://aur.archlinux.org/packages/ofono)                                                                                                |
| [netctl](https://wiki.archlinux.org/title/Netctl)                     | [netctl](https://man.archlinux.org/man/netctl.1)         | [wifi-menu](https://man.archlinux.org/man/wifi-menu.1) | No                                                                           | PPPoE and Mobile broadband support via [ppp](https://archlinux.org/packages/core/x86_64/ppp/)                                                                                                   |
| [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)     | [nmcli](https://man.archlinux.org/man/nmcli.1)           | [nmtui](https://man.archlinux.org/man/nmtui.1)         | Yes                                                                          | PPPoE supported via [rp-pppoe](https://archlinux.org/packages/extra/x86_64/rp-pppoe/). Mobile broadband supported via [modemmanager](https://archlinux.org/packages/extra/x86_64/modemmanager/) |
| [sustemd.networkd](https://wiki.archlinux.org/title/Systemd-networkd) | [networkctl](https://man.archlinux.org/man/networkctl.1) | No                                                     | No                                                                           | Only support Ethernet connection                                                                                                                                                                |
| [wpa_supplicant](https://wiki.archlinux.org/title/Wpa_supplicant)     | [wpa-cli](https://man.archlinux.org/man/wpa_cli.8)       | No                                                     | [wpa_supplicant_gui](https://aur.archlinux.org/packages/wpa_supplicant_gui/) | Only support Ethernet [802.1X](https://en.wikipedia.org/wiki/IEEE_802.1X)                                                                                                                       |
| [iwd](https://wiki.archlinux.org/title/Iwd)                           | [iwctl](https://man.archlinux.org/man/iwctl.1)           | No                                                     | [iwgtk](https://aur.archlinux.org/packages/iwgtk/)                           | Only support Ethernet [802.1X](https://en.wikipedia.org/wiki/IEEE_802.1X)                                                                                                                       |

### Initramfs
Creating a new _initramfs_ is usually not required because [mkinitcpio](https://wiki.archlinux.org/title/Mkinitcpio) 
was already run buy _pacstrap_ during kernel install. If you are using LVM, system encryption or RAID you need to
modify [mkinitcpio.conf](https://man.archlinux.org/man/mkinitcpio.conf.5) with appropriate settings and then recreate
the image with:
```shell
mkinitcpio -P
```

### Boot loader
This is the last configuration step. We need to install a Linux-capable bootloader. If you have an Intel or AMD CPU is
also suggested to install their [microcode](https://wiki.archlinux.org/title/Microcode). For better chose
the appropriate bootloader we suggest to follow the 
[ArchWiki page](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader). This part does not differ from the 
original one.


## Reboot and have fun
We have finally done it! ArchLinux32 is installed. Now we need to exit for chroot, unmount all partitions and then
reboot to the installed os. To exit chroot you need to type `exit` or press `CTRL+d`.

After we have returned to the installation environment we can unmount all partition by referring their mount point.
For example to unmount root we need to use this command:
```shell
umount -R /mnt
```

Once we have unmounted **ALL** the partition we can reboot with `reboot` command and start to enjoy the new fresh
ArchLinux32 install.


<seealso>
    <category ref="arc32">
        <a href="https://wiki.archlinux.org/title/Installation_guide">Installation Guide - ArchWiki</a>
    </category>
</seealso>