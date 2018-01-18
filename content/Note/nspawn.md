---
title: "2018-01-18-install-a-centos-container(rootfs)-on-centos"
date: 2018-01-18 14:40
---

this Note is about how to install a centos root filesystem on centos and run it with systemd-nspwan as a container 

###create a directory to install centos in
```
#mkdir /centos_chroot
#mkdir -p /centos_chroot/var/lib/rpm
```

###create the rpm database
```
#rpm --rebuilddb --root=/centos_chroot/
```

###get centos-release-{version}.el7.centos.2.10.x86_64.rpm from network

may from [ here ](https://centos.pkgs.org/7/centos-x86_64/centos-release-7-4.1708.el7.centos.x86_64.rpm.html) or [ here ](http://rpm.pbone.net/index.php3/stat/4/idpl/31977512/dir/centos_7/com/centos-release-7-2.1511.el7.centos.2.10.x86_64.rpm.html)

###use rpm and yum to install centos to centos_chroot
```
#rpm -i --root=/centos_chroot --nodeps centos-release-{version}.centos.2.10.x86_64.rpm
#yum --installroot=/centos_chroot groups install -y -q 'Minimal Install'
```

###Set the password for the root user
```
#chroot /centos_chroot passwd root //make sure the host's selinux is closed
```

###boot the centos via systemd-nspawn
```
#systemd-nspawn -D /centos_chroot -b
```

then just like work in a virtual mechine

###kill the container
in another window 
```
#machinectl poweroff centos_chroot
``` 

###production

just package the centos_chroot directory and copy the package to the target machine, unpackage it,and then:

```
#systemd-nspawn -D /centos_chroot -b
``` 

everything is almost done

###issues

1. the package maybe large, when installed centos, centos_chroot take up 1.3G storage, after installed papi, mysql and nginx, it take up 2.3G.
2. the container and the host share the same network defaultly(also may be configuraed to use NAT or brige), need take case the conflict of network ports.
3. container are limit to modify some fs,such as proc.
4. there maybe other unknown subtle tricks. 
