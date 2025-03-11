# ubuntu-mainline-kernel.sh

Bash script for Ubuntu (and derivatives such as LinuxMint) to easily (un)install kernels from the [Ubuntu Kernel PPA](https://kernel.ubuntu.com/~kernel-ppa/mainline/).

## Warnings

:warning: Use this script at your own risk. Be aware that the kernels installed by this script are [unsupported](https://wiki.ubuntu.com/Kernel/MainlineBuilds#Support_.28BEWARE:_there_is_none.29).

:unlock: Do not use this script if you don't need to or don't know what you're doing. You won't be [covered](https://github.com/pimlie/ubuntu-mainline-kernel.sh/issues/32) by any security guarantees.
Ubuntu's intended use for the mainline ppa kernels is for debugging.

:information_source: We strongly recommend that you leave the default Ubuntu kernel installed, as there is no guarantee that you will have at least one kernel installed on your system.

## Installing
```
apt install wget
wget https://raw.githubusercontent.com/pimlie/ubuntu-mainline-kernel.sh/master/ubuntu-mainline-kernel.sh
chmod +x ubuntu-mainline-kernel.sh
sudo mv ubuntu-mainline-kernel.sh /usr/local/bin/
```

If you want to automatically check for a new kernel version when you log in:
```
wget https://raw.githubusercontent.com/pimlie/ubuntu-mainline-kernel.sh/master/UbuntuMainlineKernel.desktop
mv UbuntuMainlineKernel.desktop ~/.config/autostart/
```

## SecureBoot

> :warning: There is no support for creating and registering your own MOK. If you don't know how to do this, you can use the `mok-setup.sh` script from
> [berglh/ubuntu-sb-kernel-signing](https://github.com/berglh/ubuntu-sb-kernel-signing) to get started (at your own risk).

The script supports mainline kernel self-signing. Edit the script and add `sign_kernel=1` and update the paths to your MOK key & certificate.
(The default paths are those created by the `mok-setup.sh` script from [berglh/ubuntu-sb-kernel-signing](https://github.com/berglh/ubuntu-sb-kernel-signing)).

## Usage
```
Usage: ubuntu-mainline-kernel.sh -c|-l|-r|-u

Download and install the latest kernel available from kernel.ubuntu.com

Arguments:
  -c               Check if a newer kernel version is available
  -b [VERSION]     Build and install kernel VERSION locally (requires git & docker)
  -i [VERSION]     Install kernel VERSION, see -l for list. You don't need to prefix
                   with v. For example, -i 4.9 is the same as -i v4.9. If version is omitted
                   is omitted, the latest available version will be installed.
  -l [SEARCH]      List locally installed kernel versions. If an argument is given to this
                   option, it will search for it.
  -r [SEARCH]      List available kernel versions. If an argument is given to this option
                   is given, it will search for them.
  -u [VERSION]     Uninstall the given kernel version. If version is omitted,
                   a list of up to 10 installed kernel versions will be displayed.
  --update         Update this script by re-downloading it from github
  -h               Show this message

Optional:
  -p, --path DIR       The working directory, .deb files will be downloaded to this
                       folder. If omitted, the folder /tmp/ubuntu-mainline-kernel.sh/ will be used.
                       will be used. Path is relative to $PWD
  -ll, --low-latency   Use the low-latency version of the kernel, amd64 & i386 only
  -lpae, --lpae        Use the Large Physical Address Extension kernel, armhf only
  --snapdragon         Use the Snapdragon kernel, arm64 only
  -do, --download-only Download the deb files only, do not install them
  -ns, --no-signature  Do not check the gpg signature of the checksum file
  -nc, --no-checksum   Do not check the sha checksums of the .deb files
  -d, --debug          Show debug information, all internal commands will echo their output
  --rc                 Include release candidates
  --yes                Assume yes to all questions (use with care!)
```

> :information_source: As of ~v5.18, Ubuntu no longer releases low-latency mainline kernels, see this
> [AskUbuntu](https://askubuntu.com/questions/1397410/where-are-latest-mainline-low-latency-kernel-packages) for more information.

## Elevated privileges

This script requires elevated privileges when installing or uninstalling kernels.

Either run this script with sudo, or configure the path to sudo within the script to sudo automatically.

## Building kernels locally *(EXPERIMENTAL)*

> :warning: YMMV, this is experimental support. Do not build kernels unless you know what you're doing.

> :warning: If the build fails, please debug yourself and file a PR with fixes if necessary.
> Also, if you don't know how to debug the build failure, you probably shouldn't be building your own kernels!

> :information_source: There are no plans to add full support for building kernels. This functionality may remain experimental for a long time.

The mainline kernel ppa only supports the latest Ubuntu release. But newer Ubuntu releases might use newer library versions than the current LTS releases
(e.g. both libssl and glibc version issues have existed in the past). This means that you won't be able to (fully) install the newer kernel.

If this happens, you can try to build your own kernel versions using the `--build VERSION` argument (e.g. `-b 6.7.0`).

Kernel building support is provided by [TuxInvader/focal-mainline-builder](https://github.com/TuxInvader/focal-mainline-builder), so this is required:

- git & docker
- quite a bit of free disk space (~3GB to check out the kernel sources, maybe ~10GB or more during build)
- may take quite a while, depending on how fast your machine is

## Example output

Install the latest version:
```
 ~ $ sudo ubuntu-mainline-kernel.sh -i
Finding latest version available on kernel.ubuntu.com
Latest version is v4.9.0 but seems its already installed, continue? (y/N)
Will download 5 files from kernel.ubuntu.com:
CHECKSUMS
CHECKSUMS.gpg
linux-headers-4.9.0-040900-generic_4.9.0-040900.201612111631_amd64.deb
linux-headers-4.9.0-040900_4.9.0-040900.201612111631_all.deb
linux-image-4.9.0-040900-generic_4.9.0-040900.201612111631_amd64.deb
Signature of checksum file has been successfully verified
Checksums of deb files have been successfully verified with sha256sum
Installing 3 packages
[sudo] password for pimlie:
Cleaning up work folder
```
Uninstalling a version from a list
```
 ~ $ sudo ubuntu-mainline-kernel.sh -u
Which kernel version do you wish to uninstall?
[0]: v4.8.6-040806
[1]: v4.8.8-040808
[2]: v4.9.0-040900
type the number between []: 0
Are you sure you wish to remove kernel version v4.8.6-040806? (y/N)
The following packages will be removed:
linux-headers-4.8.6-040806-generic:amd64 linux-headers-4.8.6-040806-generic:all linux-image-4.8.6-040806-generic:amd64
Are you really sure? (y/N)
[sudo] password for pimlie:
Kernel v4.8.6 successfully purged
```

## Dependencies

* bash
* gnucoreutils
* dpkg
* wget (as of 2018-12-14 as the kernel ppa is now https only)

## Optional dependencies

* libnotify-bin (to show notification bubble when new version is found)
* bsdmainutils (format output of -l, -r with column)
* gpg (to check the signature of the checksum file)
* sha1sum/sha256sum (to check .deb checksums)
* sbsigntool (to sign kernel images for SecureBoot)
* sudo

## Known issues (with workarounds)
- GPG is unable to import the key behind a proxy: [#74](https://github.com/pimlie/ubuntu-mainline-kernel.sh/issues/74)
