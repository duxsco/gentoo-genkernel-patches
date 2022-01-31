# genkernel-patches

These patches allow for the decryption of multiple LUKS encrypted root and swap devices. They are based upon:
https://bugs.gentoo.org/694778

Patch 02 is optional. It has been created in addition and allows for the [remote decryption over ssh](https://github.com/duxsco/gentoo-installation#remote-unlock) with the use of [dosshd](https://wiki.gentoo.org/wiki/Handbook:AMD64/Blocks/Booting).

Step by step instructions can be found at https://github.com/duxsco/gentoo-installation#kernel

  1. Store the files under:

```bash
$ tree /etc/portage/patches/sys-kernel/genkernel
/etc/portage/patches/sys-kernel/genkernel
├── 00_defaults_linuxrc.patch
├── 01_defaults_initrd.scripts.patch
└── 02_defaults_initrd_dosshd.scripts.patch

0 directories, 2 files
$ chown -R root: /etc/portage/patches
```

  2. Install `sys-kernel/genkernel` and make sure that ` * User patches applied.` is printed out:

```bash
$ sudo -i emerge -av genkernel
Password: 

These are the packages that would be merged, in order:

Calculating dependencies... done!
[ebuild   R    ] sys-kernel/genkernel-4.2.3::gentoo  USE="firmware (-ibm)" PYTHON_SINGLE_TARGET="python3_9 (-python3_10) -python3_8" 0 KiB

Total: 1 package (1 reinstall), Size of downloads: 0 KiB

Would you like to merge these packages? [Yes/No] yes
>>> Verifying ebuild manifests
>>> Emerging (1 of 1) sys-kernel/genkernel-4.2.3::gentoo
>>> Installing (1 of 1) sys-kernel/genkernel-4.2.3::gentoo
>>> Jobs: 1 of 1 complete                           Load avg: 1.08, 0.63, 0.42

 * Messages for package sys-kernel/genkernel-4.2.3:

 * User patches applied.
>>> Auto-cleaning packages...

>>> No outdated packages were found on your system.

 * GNU info directory index is up-to-date.
```

  3. Add the following lines to `/etc/default/grub` (add your own UUIDs). You can use `crypt_roots` and `crypt_swaps` as often as you like.

```
MY_CRYPT_ROOT="crypt_roots=UUID=28dc484d-045f-4e62-8244-a3710927878e crypt_roots=UUID=ef880bbc-0774-45cb-add3-378ab4a19a0c root_key=key"
MY_CRYPT_SWAP="crypt_swaps=UUID=14fa29a5-7d22-4095-9295-4a5520e76403 crypt_swaps=UUID=b2b148aa-63ca-42ca-8132-49e18a95b81b swap_key=key"
GRUB_CMDLINE_LINUX_DEFAULT="${MY_CRYPT_ROOT} ${MY_CRYPT_SWAP} keymap=de"
GRUB_ENABLE_CRYPTODISK="y"
```

  4. I recommend the use of a key file provided the initramfs which contains the key file is stored on an encrypted boot partition. You need to set kernel options `root_key=key` and `swap_key=key` for the key file to be used:

```
$ sudo -i mkdir -p /key/mnt/key
$ sudo -i bash -c "(umask 0377 && dd bs=512 count=16384 iflag=fullblock if=/dev/random of=/key/mnt/key/key)"
$ sudo -i cryptsetup luksAddKey --key-file /key/mnt/key/key /dev/firstRoot
etc.
```
  5. `INITRAMFS_OVERLAY="/key"` must be set in `/etc/genkernel.conf`.

## Other Gentoo Linux repos

https://github.com/duxsco?tab=repositories&q=gentoo-
