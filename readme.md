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

For filesystem user space utilities, consider using [ntfsprogs-plus](https://github.com/ntfsprogs-plus/ntfsprogs-plus) which is maintained by the same maintainer of this driver. 
A packaged version of these utilities will be provided in this repository in a future release.

## License

Copyright notices and license information remain in the original source files.

