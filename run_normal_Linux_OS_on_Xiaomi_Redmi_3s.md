# Run normal GNU/Linux OS on Xiaomi Redmi 3s (Qualcomm Snapdragon 430 msm8937)
(not yet)

by Yangyangnau

(I'm not a native English speaker. Sorry for my English.)

##Xiaomi hasn't release Redmi 3s kernel to us. It's SHAME.(2016.08.27)

I want run normal GNU/Linux OS(debian gentoo ...) on Xiaomi Redmi 3s without chroot and virtual machine, and get all hardware working with libhybris perhaps.
So I am try porting QAEP (msm8937) kernel to Redmi 3s.

**Years ago, I had run debian sid on an android TV with android in a chroot env.**

It's simple:

Bootstrap a armeabi rootfs on debian host pc,

chroot in it  with `qemu-arm-static`'s help,

install a kernel, get a initrd.img(here I use dracut, modfy it, hardcode right boot cmdline to it),

using u-boot's `mkbootimg` pack initrd.img to android's boot.timg with
 android's stock kernel,
 
add a andorid's emmc partition(not easy or using a sd card) for debian rootfs,

copy android's stock kernel modules(in android's system) to new rootfs,

run `depmod -a` and add to new rootfs's `/etc/modules`,

unpack android's boot.img get android's initrd.img and uncpio to new rootfs's dir `/sysroot/android` and add `chroot /sysroot/android /init` to `/etc/rc.local`,

adjust new rootfs, disable udev and network and other unwant stuff,

flash new boot.img and new rootfs to TV,

done, that's it. 

**Yes, android's init can run normally as pid x(x>1).**

## 1. Downlaod QAEP (msm8937), get kernel source.

https://developer.qualcomm.com/download/db410c/linux-android-software-build-and-installation-guide.pdf [Link](https://developer.qualcomm.com/download/db410c/linux-android-software-build-and-installation-guide.pdf)

https://wiki.codeaurora.org/xwiki/bin/QAEP/ [Link](https://wiki.codeaurora.org/xwiki/bin/QAEP/)

https://wiki.codeaurora.org/xwiki/bin/QAEP/release [Link](https://wiki.codeaurora.org/xwiki/bin/QAEP/release)

>June 23, 2016 LA.UM.5.3.1-01010-89xx.0 msm8937_64 LA.UM.5.3.1-01010-89xx.0.xml 06.00.01

>June 07, 2016   LA.UM.5.1-04210-8x37.0          msm8937_64     LA.UM.5.1-04210-8x37.0.xml       06.00.01

```
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
. ~/.bashrc

mkdir -p ~/bin
wget -c https://storage.googleapis.com/git-repo-downloads/repo
mv repo ~/bin

mkdir -p android-source/qaep
cd android-source/qaep

repo init -u git://codeaurora.org/platform/manifest.git -b release -m LA.UM.5.3-01110-8x37.0.xml --repo-url=git://codeaurora.org/tools/repo.git
#repo init -u git://codeaurora.org/platform/manifest.git -b release -m LA.UM.5.1-04210-8x37.0.xml --repo-url=git://codeaurora.org/tools/repo.git
# repo init -u git://codeaurora.org/platform/manifest.git -b release -m LA.UM.5.3.1-01010-89xx.0.xml --repo-url=git://codeaurora.org/tools/repo.git
# repo init -u git://codeaurora.org/platform/manifest.git -b release -m LA.UM.5.3.1-01010-89xx.0.xml
# repo init -u git://codeaurora.org/platform/manifest.git -b release -m LA.UM.5.3.1-01010-89xx.0.xml --depth=1
# repo forall -c git gc
# repo sync --current-branch --force-sync --no-clone-bundle
# repo sync --current-branch -f --force-sync --no-clone-bundle -j3
repo sync -f -j3
repo sync -f
```

## 1'. Don't  want download full QAEP, here is some patch files.

patch-linux-3.18.20-to-qaep_msm8937-2016.02.25-LA.UM.5.3-01110-8x37.0.xz [link](https://github.com/yangyangnau/tips/raw/master/patch/patch-linux-3.18.20-to-qaep_msm8937-2016.02.25-LA.UM.5.3-01110-8x37.0.xz)

patch-linux-3.18.20-to-qaep_msm8937-2016.05.15-LA.UM.5.1-04010-8x37.0.xz [link](https://github.com/yangyangnau/tips/raw/master/patch/patch-linux-3.18.20-to-qaep_msm8937-2016.05.15-LA.UM.5.1-04010-8x37.0.xz)

patch-linux-3.18.24-to-qaep_msm8937-2016.06.23-LA.UM.5.3.1-01010-89xx.0.xz [link](https://github.com/yangyangnau/tips/raw/master/patch/patch-linux-3.18.24-to-qaep_msm8937-2016.06.23-LA.UM.5.3.1-01010-89xx.0.xz)

```
wget -c https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.18.20.tar.xz
wget -c https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.18.20.tar.sign

gpg --keyserver hkp://keys.gnupg.net --recv-keys 6092693E
xzcat linux-3.18.20.tar.xz | gpg --verify linux-3.18.20.tar.sign -

tar -xvf linux-3.18.20.tar.xz
mv linux-3.18.20 linux-3.18.20-qaep_msm8937-2016.02.25-LA.UM.5.3-01110-8x37.0
cd linux-3.18.20-qaep_msm8937-2016.02.25-LA.UM.5.3-01110-8x37.0
xzcat ../patch-linux-3.18.20-to-qaep_msm8937-2016.02.25-LA.UM.5.3-01110-8x37.0.xz | patch -p1
```

