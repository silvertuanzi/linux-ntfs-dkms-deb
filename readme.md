# NTFS driver for debian

This repository ports [namjaejeon/linux-ntfs](https://github.com/namjaejeon/linux-ntfs) driver to Debian/Ubuntu. 
Files in `src/` are derived from upstream commit [640b7dd](https://github.com/namjaejeon/linux-ntfs/commit/640b7dde5def187d41042ec29a66fcbc46cdcc3a), with only `src/Makefile` modified for DKMS packaging.

This driver can be built in Linux 6.1+, and is included in Linux mainline 7.1+. 

## Installation

Firstly, make sure kernel headers are installed. In Debian it usually is `linux-headers-amd64`, and in Ubuntu it usually is `linux-headers-generic` or `linux-headers-generic-hwe-xx.04`. 

Find the installed kernels and headers:
```bash
apt list --installed | grep -E "linux-image|linux-headers"
```
If missing headers, install it. Make sure headers match your running kernel:
```bash
uname -a
```

Then download the latest release and install:
```bash
sudo dpkg -i ntfs-dkms_*.deb
```

The package will automatically register the driver with DKMS and build it for installed kernels.

## Mount options

This driver provides two new options for symlinks compat.
1. `symlink=<native|wsl>`:   
	`wsl(default)`: When create symlinks, the symlinks can't be read by Windows native and Linux ntfs3 driver and can only be read by wsl. Faster.   
	`native`: When create symlinks, the symlinks can be read by Windows native and Linux ntfs3 driver, and should perform the same as symlinks created on Windows.  
2. `native_symlink=<raw|rel>`:  
	`raw(default)`: The absolute target path (ni->target) is returned as-is without translation.  
	`rel`: ntfs_translate_junction() is called to rewrite the absolute path as a relative path anchored at the volume root.  

## Notes

This driver registers itself as the `ntfs` filesystem type.
If `ntfs-3g` is also installed, `mount -t ntfs` may select a different driver than expected. To avoid ambiguity, uninstalling `ntfs-3g` is recommended.

For filesystem user space utilities, consider using [ntfsprogs-plus](https://github.com/ntfsprogs-plus/ntfsprogs-plus) which is maintained by the same upstream maintainer as this driver.
Debian packaging is available at [here](https://github.com/silvertuanzi/ntfsprogs-plus-debpackage).

> [!TIP]
> If `/usr/sbin/mount.ntfs` exists, `mount -t ntfs` will always call it first instead of mounting the filesystem directly, and since this file is provided by `ntfs-3g`, it will always use the `ntfs-3g` driver.  
> If you want to use this new driver and keep `ntfs-3g` installed, use `mount -i -t ntfs` to ignore the mount helper, and this kernel ntfs driver will be used instead.  
> A mount helper can be installed as `/sbin/mount.ntfsplus`:  
> ```sh
> #!/bin/sh
> exec /usr/bin/mount -i -t ntfs "$@"
> ```
> Give it execute permission: `chmod +x /sbin/mount.ntfsplus`.  
> Then you can use this driver by `mount -t ntfsplus` to avoid conflict with `ntfs-3g`.  
> This approach is still experimental, and I'm considering packaging it in a separate package after further testing.  
> Alternatively, in this way, you can also configure `udisks2` by editing `/etc/udisks2/mount_options.conf`: add `ntfsplus` to `ntfs_drivers`, then define `ntfs:ntfsplus_defaults=` and `ntfs:ntfsplus_allow=` to manage mount options.  
> An example:
> ```
> ntfs_drivers=ntfsplus,ntfs3,ntfs
> ntfs:ntfsplus_defaults=uid=$UID,gid=$GID,windows_names,symlink=native
> ntfs:ntfsplus_allow=uid=$UID,gid=$GID,umask,dmask,fmask,locale,norecover,ignore_case,windows_names,compression,nocompression,big_writes,symlink
> ```
> Now GUI file managers (Nautilus, Dolphin, ...) will use this driver first.


## License

Copyright notices and license information remain in the original source files.

