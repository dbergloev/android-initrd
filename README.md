# Android x86 Initrd

I downloaded RemixOS to try it out as a media center OS. Once booted it works fine, although encryption is support is broken and the boot script used seams like something that was put together in under 30 min, most of which is done by the x86 project. 

There are a few variables that you can apply to your boot args like choosing a partition/image as system or data. But you will have to use complete dev paths for this, so forget about installing this on an external drive. Also the installer simply wipes the first disk from the list of the `blkid` command during install mode. And for some reason the boot script had been built to manage all mount points? Android is fully capable of handling this via it's own `init`, which is also the largest reason why encryption is out of the question. If there should be any chance of getting encryption to work, Android will need to fully manage it's partitions. 

This project is an Android x86 compatible initrd project. It's the beginning of full and proper Android support for any x86 based devices. The boot script has been completly rewritten to properly manage partitions by letting Android deal with it. The script's job is locating the partitions and inform Android which ones to use for what purpose. It supports defining partitions using dev paths, image paths, uuid, label and possibly partlabel and partuuid whenever it is supported by the kernel being used. This enables the usage of external disk support since it does not entirely depend on dev paths, which may differ depending on the device you plug it into. Encryption is partially working so far. Data can be encrypted without issues, but at the moment Android refuses to decrypt it during boot and insists on a factory reset. But it's one step closer than and proves that it's not impossible. 


## Boot Parameters 

The following arguments can be added to the boot parameters in order to control the script

* _CHRDST_: Defines the path where Android's ramdisk will be mounted to __(The chroot from where Android will run)__. Default is `/chroot`. 
* _SRCDST_: Defines the path where the source/boot partition will be mounted to. This is the partition that contains the kernel and other useful resources. Defaults to `/mnt`. 
* _ROOT_: The source/boot device that should be mounted to `SRCDST`. See [Device Identifiers](#Device-Identifiers). Defaults to auto-search for any partition containing the kernel that was booted. 
* _RAMDISK_: The ramdisk device that should be mounted to `CHRDST`. See [Device Identifiers](#Device-Identifiers). Default to `SRCDST/ramdisk.img`. 
* _SYSTEM_: The system device that should be mounted to `CHRDST/system`. See [Device Identifiers](#Device-Identifiers). Default to `SRCDST/system.img`.
* _DATA_: The data device that should be mounted to `CHRDST/data`. See [Device Identifiers](#Device-Identifiers). Defaults to `tmpfs`. 
* _CACHE_: The cache device that should be mounted to `CHRDST/cache`. See [Device Identifiers](#Device-Identifiers). Defaults to `tmpfs`.
* _FSTAB_: Alternative fstab file that will replace the one in the defined ramdisk. Defaults to `SRCDST/fstab`.
* _AUTOMOUNT_: True/False _(`1/0`)_. If true _(`1`)_ all partitions like system, data and cache will be mounted by the boot script. Otherwise the fstab file contained within the ramdisk will be updated to match selected options. 
* _DEBUG_: True/False _(`1/0`)_. If true _(`1`)_ the boot script will run in debug mode. This enables TTY support with access to the main file system _(`initrd`)_ even after Android has booted. It also starts a debug terminal before Android is started. 
* _HEADER_: Default crypt header file. Activates data encryption, sadly decryption during boot does not currently work. Defaults to empty, encryption support will be disabled. 


## Device Identifiers

The boot script offers several ways to identify a device __(partition, image etc)__. 

* _Dev Path_: Regular dev path like `/dev/sdb1`. 
* _File Path_: Path to an image file relative to `SRCDST`. This can either be a `gz` compressed `cpio` file that will be extracted or a file system image that will be mounted. WARNING: Any `gz` compressed `cpio` file will be extracted to tmpfs filesystems, e.g. memory. This is fine for things like a ramdisk, but do not attempt this with a large system image. Despite the fact that it would take time to extract, it might also end up requiring all of your available memory. 
* _UUID_: Identify a partition based on it's `uuid`. This requires you to define the argument like `DATA=UUID=<id>`. 
* _PARTUUID_: Identify a partition based on it's `partuuid`. This requires you to define the argument like `DATA=PARTUUID=<id>`. WARNING: This is likely not supported by the kernel. Make sure of it's support before using it. 
* _LABEL_: Identify a partition based on it's `label`. This requires you to define the argument like `DATA=LABEL=<id>`.
* _PARTLABEL_: Identify a partition based on it's `partlabel`. This requires you to define the argument like `DATA=PARTLABEL=<id>`. WARNING: This is likely not supported by the kernel. Make sure of it's support before using it. 


## Example

```
menuentry 'Android' {
	search --file --no-floppy --set=root /kernel
	linux /kernel root=/dev/ram0 ROOT=LABEL=BOOT RAMDISK=/ramdisk.img SYSTEM=/dev/sda2 DATA=UUID=602f4b54-0115 DEBUG=1
	initrd /initrd.img
}
```

