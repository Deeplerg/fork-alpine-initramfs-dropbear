# alpine-initramfs-dropbear
unlock cryptdisk remotely

## About

Build Alpine Linux initramfs with dropbear to unlock an encrypted root-disk remotely via ssh.
The `initramfs-init-dropbear`-init-script is a slightly modified version of
the default-init which comes with Alpine Linux 3.20.2

## Requirements
Requires dropbear and cryptsetup. Assumes you already have a working cryptsetup setup, i.e. are able
to decrypt root at boot.

## Installation

```
git clone https://github.com/Deeplerg/fork-alpine-initramfs-dropbear
cd alpine-initramfs-dropbear
```

```
cp features.d/* /etc/mkinitfs/features.d/
cp initramfs-init-dropbear /usr/share/mkinitfs/initramfs-init-dropbear
cp dropbear/unlock_disk /etc/dropbear/
```

- Add `dropbear` (and `network` if not present – also `virtio` if network devices are virtualized) to `/etc/mkinitfs/mkinitfs.conf`

- Copy a ssh-public-key to `/etc/dropbear/authorized_keys`
- Optional: Copy `dropbear_*_host_keys` to `/etc/dropbear` if needed, dropbear will generate
missing keys on its own. See `man dropbear` and `man dropbearkey`.

- Modify and append the following to your bootloaders kernel cmdline (mind the number of colons!):
```
dropbear=<port> cryptdm=<cryptdm> cryptroot=<crypt-dev> ip=<ip>::<gw>:<mask>::<interface>
```
For `ip=` see https://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt

e.g for extlinux you should modify ``/etc/update-extlinux.conf``:
```
modules=sd-mod,usb-storage,ext4,ata_piix,e1000
```
```
default_kernel_opts="quiet rootfstype=ext4 dropbear=5555 cryptroot=/dev/sda2 cryptdm=crypt ip=1.2.3.4::1.2.3.1:255.255.255.0::eth0"
```

It is crucial, that all modules needed for disk-decryption and networking are listed in the
`modules=`-list!

If you're dealing with a VPS, you may also need to add `virtio_net` to the `modules=` list.

- update bootloader, e.g for extlinux:
```
update-extlinux
```
- build initramfs:
```
mkinitfs -i /usr/share/mkinitfs/initramfs-init-dropbear
```
- reboot

If something goes wrong, remove `dropbear=<port>` in your
bootloader-prompt, and you should be asked for your password as before.
