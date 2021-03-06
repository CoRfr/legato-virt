# legato-virt
Documentation for the Legato virtual platform.

This qemu-based solution allows you to emulate a Legato target on a computer.
Most of the software runs just like on real hardware (such as a WP85xx on a [MangOH](http://mangoh.io/)).
However, some parts (Modem services, Positioning services, ...) are simulating things and either returning pre-defined values,
or values that can be configured through a different set of APIs.

## Quick Start

### Run from Docker

The simplest way to run the target is through [Docker](https://www.docker.com/):
```
$ docker run -ti quay.io/legato/virt-x86
...
Poky (Yocto Project Reference Distro) 2.2.3 swi-virt-x86 /dev/ttyS0

swi-virt-x86 login:
```
(default login is `root`, no password, and runs the latest release)

The containers exposed an SSH server, that you can eventually expose using `-p` option:
```
$ docker run -d -p 10022:22 quay.io/legato/virt-x86
```
This allows you to connect to target like this:
```
$ ssh -p 10022 root@localhost
root@swi-virt-x86:~#
```

The repository also contains tagged versions of Legato:
```
$ docker run -ti -p 10022:22 quay.io/legato/virt-x86:17.09.0
```
(to run version `17.09.0`)

and contains a 'nightly' branch that is rebuilt with the latest code from master branch every day:
```
$ docker run -ti -p 10022:22 quay.io/legato/virt-x86:nightly
```

### Run from Legato

To build the target from Legato, you will need to get a toolchain.

Latest toolchain for `x86` variant, `LXSWI2.2-6.0`, is available at:
- 32-bit host: [poky-swi-glibc-i686-meta-toolchain-swi-i586-toolchain-swi-LXSWI2.2-6.0+virt.sh](http://downloads.sierrawireless.com/legato/virt/LXSWI2.2-6.0/x86/sdk/poky-swi-glibc-i686-meta-toolchain-swi-i586-toolchain-swi-LXSWI2.2-6.0+virt.sh)
- 64-bit host: [poky-swi-glibc-x86_64-meta-toolchain-swi-i586-toolchain-swi-LXSWI2.2-6.0+virt.sh](http://downloads.sierrawireless.com/legato/virt/LXSWI2.2-6.0/x86/sdk/poky-swi-glibc-x86_64-meta-toolchain-swi-i586-toolchain-swi-LXSWI2.2-6.0+virt.sh)

Latest toolchain for `arm` variant, `LXSWI2.2-6.0`, is available at:
- 32-bit host: [poky-swi-glibc-i686-meta-toolchain-swi-armv5e-toolchain-swi-LXSWI2.2-6.0+virt.sh](http://downloads.sierrawireless.com/legato/virt/LXSWI2.2-6.0/arm/sdk/poky-swi-glibc-i686-meta-toolchain-swi-armv5e-toolchain-swi-LXSWI2.2-6.0+virt.sh)
- 64-bit host: [poky-swi-glibc-x86_64-meta-toolchain-swi-armv5e-toolchain-swi-LXSWI2.2-6.0+virt.sh](http://downloads.sierrawireless.com/legato/virt/LXSWI2.2-6.0/arm/sdk/poky-swi-glibc-x86_64-meta-toolchain-swi-armv5e-toolchain-swi-LXSWI2.2-6.0+virt.sh)

Install this toolchain by running the script.
It installs it by default in `/opt/swi/LXSWI2.2-6.0+virt` but you can eventually install it somewhere else such as `$HOME/legato/sdk/LXSWI2.2-6.0+virt`.
In this case, you will need to define the following environment variable:
```
HOST_ARCH=$(uname -m)
VIRT_X86_TOOLCHAIN_DIR=$HOME/legato/sdk/LXSWI2.2-6.0+virt/sysroots/$HOST_ARCH-pokysdk-linux/usr/bin/i586-poky-linux
VIRT_ARM_TOOLCHAIN_DIR=$HOME/legato/sdk/LXSWI2.2-6.0+virt/sysroots/$HOST_ARCH-pokysdk-linux/usr/bin/armv5e-poky-linux
```

To checkout the Legato source code, follow instructions from [legato-af/README.md](https://github.com/legatoproject/legato-af#clone-from-github).

Then build:
```
export VIRT_TARGET_ARCH=x86
# OR
export VIRT_TARGET_ARCH=arm
$ make virt
```

And run:
```
$ . bin/configlegatoenv
$ export IMG_MIRROR_LINUX=http://downloads.sierrawireless.com/legato/virt/LXSWI2.2-6.0/${VIRT_TARGET_ARCH}/images
$ simu start
```

Note: Declaration of `IMG_MIRROR_LINUX` is compulsory at the moment as the default location is outdated.

By default, qemu tries to expose the SSH server running on the target on port `10022`.

## Platform info

It currently exists in 2 variants:
- virt-x86
> CPU Arch: i586
>
> The fastest on a standard PC as it doesn't need as much instruction translations as with an arm emulation.
> Can also be accelerated through the usage of [KVM](https://wiki.qemu.org/Features/KVM).
- virt-arm
> CPU Arch: armv5e
>
> Closer to what is found on a hardware target.

## How to

### Install sample apps

Sample apps `.update` are already shipped with the image, and located in `/mnt/flash/samples`.

To install them, from the target run:
```
# update < /mnt/flash/samples/helloWorld.update
```

### Install apps

Regular legato tools such as `update` can communicate with the virtual platform, but an `SSH_PORT` env variable needs to be set to point to SSH port, which is `10022` by default (on the current machine, so `localhost`).

So this would look like:
```
$ export SSH_PORT=10022
$ update build/virt/samples/helloWorld.virt.update localhost
```

To update the target using a `.update`, you can simply use the `simu update [UPDATE_PATH]` command.
For instance:
```
$ simu update build/virt/samples/helloWorld.virt.update .
```

For versions of Legato `< 18.02.0`, this command does not exist.
It is safer to transfer the `.update` file to the target with scp, and then install it from the target:
```
$ simu push build/virt/samples/helloWorld.virt.update .
$ simu ssh "/legato/systems/current/bin/update < helloWorld.virt.update"
```
OR (without transfer)
```
$ simu ssh /legato/systems/current/bin/update < build/virt/samples/helloWorld.virt.update
```
