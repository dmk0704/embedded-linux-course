# Lab 01 report

- Student name: Dmytro Komarenko
- GitHub username: dmk0704
- Image filename: am335x-debian-13.6-base-v6.12-armhf-2026-07-24-4gb.img.xz
- SHA-256: 796777685b309328abcab3a122f426282c83fbde7f481ebe3e20829441f878a9 — matches the expected checksum from the lab instructions.

## Running Linux on BBB

### BBB: `uname -r`

```text
6.12.96-bone64
```

### BBB: `cat /etc/os-release`

```text
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
VERSION_ID="13"
VERSION="13 (trixie)"
VERSION_CODENAME=trixie
DEBIAN_VERSION_FULL=13.6
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

### BBB: `lsblk`

```text
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0      179:0    0 59.5G  0 disk
├─mmcblk0p1  179:1    0   36M  0 part /boot/firmware
├─mmcblk0p2  179:2    0  512M  0 part [SWAP]
└─mmcblk0p3  179:3    0   59G  0 part /
zram0        253:0    0  241M  0 disk [SWAP]
mmcblk1      179:256  0  3.6G  0 disk
└─mmcblk1p1  179:257  0  3.6G  0 part
mmcblk1boot0 179:512  0    2M  1 disk
mmcblk1boot1 179:768  0    2M  1 disk
```

The root filesystem `/` is mounted from `mmcblk0p3`, confirming that the BBB is running from the microSD card.

### BBB: `ip addr`

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever

2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 78:04:73:41:2a:50 brd ff:ff:ff:ff:ff:ff
    altname end0
    altname enx780473412a50

3: usb0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 78:04:73:41:2a:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.7.2/30 brd 192.168.7.3 scope global usb0
       valid_lft forever preferred_lft forever
    inet6 fe80::7a04:73ff:fe41:2a53/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

The BBB USB network interface is `usb0` with IPv4 address `192.168.7.2/30`.

## First user-space program

### HOST, in this lab directory: `file build/hello`

```text
build/hello: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, BuildID[sha1]=4d218ed1709c475b9e9646c4b572d614f07c7ef1, for GNU/Linux 3.2.0, not stripped
```

This confirms that the program was cross-compiled for 32-bit ARM rather than the x86-64 HOST architecture.

### BBB: `/home/debian/labs/lab01/hello`

```text
Hello from Lab 01 on BeagleBone Black!
```

## Shared kernel workspace

### HOST: `ls -lh ~/bbb-workspace/kernel/bb-kernel/deploy`

```text
total 35M
-rw-rw-r-- 1 python python 195K Oct 11 00:07 6.12.96-bone64-dtbs.tar.zst
-rw-rw-r-- 1 python python  27M Oct 11 00:07 6.12.96-bone64-modules.tar.zst
-rwxrwxr-x 1 python python 8.3M Oct 11 00:06 6.12.96-bone64.zImage
-rw-rw-r-- 1 python python 206K Oct 11 00:06 config-6.12.96-bone64
```

### HOST: `cat ~/bbb-workspace/kernel/bb-kernel/KERNEL/include/config/kernel.release`

```text
6.12.96-bone64
```

Do the full HOST and BBB kernel-release strings match exactly?

Yes. The running BBB kernel reports:

```text
6.12.96-bone64
```

and the HOST kernel build tree reports:

```text
6.12.96-bone64
```

The full release strings match exactly.

Why must Lab 02 use a matching kernel tree?

An external kernel module is built against a specific configured Linux kernel.
Its ABI, symbols, configuration, and version metadata depend on that kernel
build. Therefore the kernel tree used to build the module must match the
kernel running on the BBB. Otherwise the module may be incompatible and
`insmod` can reject it.

Submit this completed report with `src/hello.c` and `Makefile`. Do not commit
images, executables, toolchains, kernel sources, or build artifacts.
