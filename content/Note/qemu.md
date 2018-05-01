---
title: "2018-05-01 use qemu"
date: 2018-05-01 18:30
---

###create disk
```
dd bs=4M count=2000 if=/dev/zero of=disk.img
mkfs.ext4 disk.img
```

###install OS to the disk
```
qemu-system-x86_64 -cdrom debain.iso -hda disk.img --enable-kvm -m 1G
```

###boot the os
```
qemu-system-x86_64 -hda disk.img -m 1G --enable-kvm
```

notice defaultly, qemu can connect to the network but icmp is broken.
