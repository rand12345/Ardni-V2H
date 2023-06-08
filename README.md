# Ardni-V2H
Image files for V2G system

Tools needed for extraction

apt install squashfs-tools u-boot-tools

```
$ sudo fdisk -l  --bytes ./eMMC.bin
Disk ./eMMC.bin: 3.7 GiB, 3909091328 bytes, 7634944 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device      Boot  Start     End Sectors       Size Id Type
./eMMC.bin1 *      3072  131071  128000   65536000 83 Linux
./eMMC.bin2 *    131072  262143  131072   67108864 83 Linux
./eMMC.bin3 *    262144  393215  131072   67108864 83 Linux
./eMMC.bin4      393216 2490367 2097152 1073741824 83 Linux
```

Partitions 1-3 contain kernal, kernel_dtb configurations, and root file system
Partition 4 contains RW filesystem containing overlay for /etc /var

Part 1-3 contains a fitImage

```
$ mkdir part1
$ sudo mount eMMC.bin part1 -o offset=$((3072*512)),sizelimit=65536000
$ dumpimage -l part1/boot/fitImage 
FIT description: Boot Image for OVO Smart Energy Devices
Created:         Thu May  2 21:36:15 2019
 Image 0 (kernel)
  Description:  kernel zImage
  Created:      Thu May  2 21:36:15 2019
  Type:         Kernel Image (no loading done)
  Compression:  uncompressed
  Data Size:    3802016 Bytes = 3712.91 kB = 3.63 MB
  Hash algo:    sha1
  Hash value:   27973bdd590f3f1b8b7e34c9faaef7641388874f
 Image 1 (fdt)
  Description:  am335x-boneblack-ovo.dtb
  Created:      Thu May  2 21:36:15 2019
  Type:         Flat Device Tree
  Compression:  uncompressed
  Data Size:    39340 Bytes = 38.42 kB = 0.04 MB
  Architecture: ARM
  Hash algo:    sha1
  Hash value:   1ce4aca684e7469f4e96c2f4969ce13a07a44266
 Image 2 (ramdisk)
  Description:  SquashFS with XZ compression
  Created:      Thu May  2 21:36:15 2019
  Type:         RAMDisk Image
  Compression:  uncompressed
  Data Size:    8413184 Bytes = 8216.00 kB = 8.02 MB
  Architecture: ARM
  OS:           Linux
  Load Address: unavailable
  Entry Point:  unavailable
  Hash algo:    sha1
  Hash value:   4051e586d4c3bd26238b7e5a6f41514462727c80
 Default Configuration: 'kernel_dtb_ovo_ramdisk'
 Configuration 0 (kernel_dtb_ovo_ramdisk)
  Description:  version 263168 (ovo-ssb-v3.1.0-67-g326a984ade prod)
  Kernel:       kernel
  Init Ramdisk: ramdisk
  FDT:          fdt
```

Extraction of kernel and config

```
$ dumpimage -T flat_dt -i ./part1/boot/fitImage -p 0 vmlinux
$ file vmlinux
0: Linux kernel ARM boot executable zImage (little-endian)

$ dumpimage -i ./part1/boot/fitImage -T flat_dt -p 1 dft
$ file dft
rootfs: Device Tree Blob version 17, size=39340, boot CPU=0, string block size=2156, DT structure block size=37128

$ dumpimage -i ./part1/boot/fitImage -T flat_dt -p 2 rootfs
$ mkdir fs
$ sudo mount rootfs ./fs
$ cd fs

```

Main executable file 

```
$ file ./usr/bin/ssb
./usr/bin/ssb: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, stripped
```

Configuration located in /etc partition 4

```
.
├── etc
│   └── default
│       └── ssb
└── var
    ├── data
    │   ├── logrotate.state
    │   ├── ssbStatus.json
    │   ├── ssbTotalEnergy
    │   ├── telemetry_cache.idx
    │   └── tempCommandOutput
    └── log
        ├── calibration.log
        ├── calibration.log.1.gz
        ├── calibration.log.2.gz
        ├── cores
        │   └── old
        ├── diagnostic.log
        ├── diagnostic.log.1.gz
        ├── diagnostic.log.2.gz
        ├── diagnostic.log.3.gz
        ├── diagnostic.log.4.gz
        ├── diagnostic.log.5.gz
        ├── fault.log
        ├── fsck-result
        ├── logs.tar.gz
        ├── messages
        ├── messages.1.gz
        ├── messages.2.gz
        ├── messages.3.gz
        ├── messages.4.gz
        ├── messages.5.gz
        ├── ssb.log
        ├── ssb.log.1.gz
        ├── ssb.log.10.gz
        ├── ssb.log.2.gz
        ├── ssb.log.3.gz
        ├── ssb.log.4.gz
        ├── ssb.log.5.gz
        ├── ssb.log.6.gz
        ├── ssb.log.7.gz
        ├── ssb.log.8.gz
        ├── ssb.log.9.gz
        ├── telemetry.log
        ├── telemetry.log.1.gz
        ├── telemetry.log.2.gz
        ├── telemetry.log.3.gz
        ├── telemetry.log.4.gz
        └── telemetry.log.5.gz

8 directories, 41 files
```

```
$ cat ./etc/default/ssb 
# for debug purposes, to stop ssb starting at boot, uncomment this line
# and run ssb manually with -D flag.
#SSB_NO_AUTORUN=1

# Custom arguments to pass to SSB
#SSB_OPTS=""

# Syslog level to use for SSB
#SSB_LOG_LEVEL=ERROR

# Custom registration url for SSB
SSB_VNET_REGISTRATION_URL="https://device-management.stg.maia.vcharge.io/api/devices/"
```

Boot log of device from UART0

```
U-Boot SPL 2017.11 (May 02 2019 - 20:36:06)
Trying to boot from MMC2
** No device specified **
Using default environment



U-Boot 2017.11 (May 02 2019 - 20:36:06 +0000), Build: jenkins-etp-Rc-prod-build-35

I2C:   ready
DRAM:  512 MiB
MMC:   OMAP SD/MMC: 0, OMAP SD/MMC: 1
** No device specified **
Using default environment

<ethaddr> not set. Validating first E-fuse MAC
Net:   cpsw, usb_ether
Hit any key to stop autoboot:  0 
switch to partitions #0, OK
mmc1(part 0) is current device
** /boot/bootstatus shorter than offset + len **
Loading /boot/fitImage from mmc:1:2
Checking FIT image at 0x89000000
Using 'kernel_dtb_ovo_ramdisk' configuration
Verifying Hash Integrity ... sha1,rsa2048:ssb_prod+ OK
description = 'version 524288 - version 8.0.0 (2020-07-28T09:28:58 2734e584ac prod)'
** /boot/bootstatus shorter than offset + len **
Loading /boot/fitImage from mmc:1:3
Checking FIT image at 0x8d000000
Using 'kernel_dtb_ovo_ramdisk' configuration
Verifying Hash Integrity ... sha1,rsa2048:ssb_prod+ OK
description = 'version 459266 - version 7.2.2 (2020-03-20T10:22:09 5fa20b7b8e prod)'
** /boot/bootstatus shorter thana prevously boted O
## Loading kernel from FIT Image at 89000000 ...
   Using 'kernel_dtb_ovo_ramdisk' configuration
   Verifying Hash Integrity ... sha1,rsa2048:ssb_prod+ OK
   Trying 'kernel' kernel subimage
     Description:  kernel zImage
     Created:      2020-07-28   9:29:00 UTC
     Type:         Kernel Image (no loading done)
     Compression:  uncompressed
     Data Start:   0x890000d4
     Data Size:   9
   erifyin Hash ntegrity... sh1+ OK
## Loading ramdisk from FIT Image at 89000000 ...
   Using 'kernel_dtb_ovo_ramdisk' configuration
   Trying 'ramdisk' ramdisk subimage
     Description:  SquashFS with XZ compression
     Created:      2020-07-28   9:29:00 UTC
     Type:      04
    Data Se:    926248 Byes = .8 Mi
     Achitecure: ARM
     S:          Liux
     Enty Poin:  unvailabl
    Hash alg:    sa1
    Hashvalue:  e9dd481e1fd2f1c69f9658ec8615f788ddd
  Veriying ash Intgrity .. sha1+ OK
## Loading fdt from FIT Image at 89000000 ...
   Using 'kernel_dtb_ovo_ramdisk' configuration
   Trying 'fdt' fdt subimage
     Description:  am335x-boneblack-ovo.dtb
     Created:      2020-07-28   9:29:00 UTC
     Type:         Flat Device T   0893aa5a
    Data Sie:    9340 Byts = 384 KiB
     Achitecure: ARM
     ash ago:    ha1
    Hash alue:  1ce4ca684e769f4e9c2f4969e13a0744266
  Verifing Hsh Interity .. sha1+ OK
   Booting using the fdt blob at 0x893aa5a4
   XIP Kernel Image (no loaoperation at range890000d 893aad4]
   Loading Ramdisk to 8f729000, end 90000000 ... OK
   Loading Device Tree to 8f71c000, end 8f7289ab ... OK

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Initializing cgroup subsys cpuset
[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Initializing cgroup subsys cpuacct
[    0.000000] Linux version 4.4.155 (jenkins@7142800250.1 0191025 (GU Toolchain or the A-pofile Archiecture 9.2-219.12 (arm9.10)) ) #1 MP Tue Jul28 09:23:14UTC 2020
[    0.000000] CPU: ARMv7 Processor [413fc082] revision 2 (ARMv7), cr=10c5387d
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT ali   0.000000]Igoring memoy range 0x8000000 - 0x8000000
[   0.000000]cma: Resered 16 MiB at0x9e800000
[    0.000000] Memory policy: Data cache witeback
[    0.00000] CPU: AllCPU(s) stated in SVC mde
[    0.000000] AM335X ES2.1 (sgx neo )
[    0.00000] P one order, mbility rouping on. Total page: 96928
[    0.000000] Kernel command line: root/dev/ram0 r
[   0.000000 PID hash tble entries 2048 (orde: 1, 8192 btes)
[    0.000000] Dentry cache hash tabl entries: 5536 (order 6, 262144 ytes)
[    0.000000oe, 469K rwdta, 1676K rdata, 300K nit, 8174K ss, 29184Kreserved, 1684 cma-resered, 0 highmem)
[    0.000000] Virtual kernel memory layout:
[    0.00000]     vctor  : 0xfff0000 - 0xfff1000      4 kB)
[   0.000000     fixmap : 0xffc0000 - 0xfff0000   (30xc0000000 - 0x000000   ( 38MB)
[    0.0000]     pkmap  : 0xbe0000 - 0xc000000   (   2MB)
[    0.00000]       .tet : 0xc000000 - 0xc06236c   (6825kB)
[    .000000]      .init : xc06b3000 -0xc06fe
[    0.000000] Running RCU self tests- xc0f71a40  (8175 kB)
[    0.000000] Hierarchical RCU implementation.
[    0.000000]  Build-time adjustment of leaf fanout to 32.
    0.00000] NR_IRQS:1 nr_irqs:1 16
[    0.000000] IRQ: Found an INTC at 0xf200000 (revsion 5.0) wth 128 intrrupts
[    0.000000] OMAP clockeventsource: time2 at 2400000 Hz
[    .000017] shed_clock: 3 bits at 2MHz, resoluion 41ns, wrps every 8478484971x_idle_n: 7963585149 ns
[    0.000084] OMAP clocksource tier1 at 2400000 Hz
[   0.000621]clocksourceprobe: no mtching clocsources foud
[    0.001428] Console: colour dumm device 80x3
[    0.03666] consle [tty0] enbled
[    0.003693] Lock dependency vMAX_LOCKEP_SUBCLASES:  8
[   0.03763] ... AX_LOCK_DEPT:         48
[    0.03788] ... MA_LCKDEP_KEYS:       819
[    0.003841] ... MAXLOCKDEP_ENTIES:     37686
[    0.003998] Calibrating delay loop... 996.14 BogMPS (lpj=490736)nt: 136 bytes
[    0.078719] pid_max: default: 3768 minimu: 301
[   0.079094] ecurity Fraework initiaized
[    0.082162] Initiaizing cgrop subsys memry24(order: 0roup subsys io
[    0.082270] Initializing cgrou sbsys devics
[    0.02407] Initilizing cgrop subsys frezer
[    0.082501] Initializing cgroup ubsys perf_vent
[    0.087432] Brought up 1 CPUsuffe coherency084083] Settingup static ientity map or 0x8800820 - 0x88002f8
[    0.087488] SMP Total of 1 pocessors ativated (99.14 BogoMIP).
[    0.087526] CPU: All CPU(s) tarted in SC mode.
[   0.091442] evtmpfs: iniialized
[    0ile_ns: 191260446275000 ns
[    0.226039] futex hash table etries: 256(order: 2, 1384 bytes)
[    0.22834] pinctrl cre: initialzed pinctr subsystem
[    0.232194] NET: Registere protocol faily 16
[   0.236844]DMA: preallcated 256 KB pool for tomic cohedle: usin governor mnu
[    0.250286] OMAP GPIO hardwae version 0.
[    0.27560] hw-breapoint: debu architectue 0x4 unsuported.
[    0.275643] omap4_sram_init:nable to alocate sramneeded tohandle errataI688
[    0.275702] omap4_sram_init:Unble to getsdma: TI DMA DMA enine driver
[    0.312315] SCSI subsystem initilied
[    031304] usbcor: registerd ne interfac driver usbf
[    0.313278] usbcore: regiteed new intrface driverhub
[    .313477] ubcore: regisered new deice driverusb
[    0.314267] oma9000.i2c: cold not findpctldev fornode /ocp/l_kup@44c0000/scm@210000pimux@800/pimux_i2c2_pin, deferrin probe
[    0.314772] pps_core: LinuxPPS APIver. 1 regstered
[   0.314809] ps_core: Sotware ver. .3.6 - Copyight 2005-207 Rodolfo iometti <iometti@lnuxhed to cocksource imer1
[    .34600] NET: Regstered protcol family2
[    0.348655] TCP established hash able entrie: 4096 (ordr: 2, 16384bytes)
[   0.34879] TCP bindhash table etries: 409 (order: 5,147456 byte)
[    0.349394] TCP: Hash tables confgureenries: 256 oder: 2, 080 bytes)
    0.34973] UDP-Liteash table ntries: 256(order: 2, 040 bytes)
[    0.350336] NET: Registeed protoco family 1
[    0.353358] rootfs image is not initramfs (junk iniver, 5 ounters avilable
[    0.436291] audit: type=2000 audit(0.420:1): initialized
[    0.44283] squashs: version .0 (2009/0/31) PhillipLougher
[    0.446074] io schedulr noop regisered
[   0.stered (dfault)
[    0.452485] omap_uart 44e09000.serial: no wakeirq for uart00 size 568
[    0.453196] 44e09000.serial: ttyO0 atMMIO 0x4409000 (irq =158, base_bud = 300000) is a OMAPUART0
[[    1.164084] omap_uart 481a8000.serial: no wakeirq for uart4
[    1.170468] 481a8000.serial: ttyO4 at MMIO 0x481a8000 (irq = 159, base_baud = 3000000) is a OMAP UART4
[    1.183771] omap_rng 48310000.rng: OMAP Random Number Generator ver. 20
[    1.222489] brd: module loaded
[    1.245105] loop: module loaded
[    1.318530] davinci_mdio 4a101000.mdio: davinci mdio revision 1.6
[    1.324997] libphy: 4a101000.mdio: probed
[    1.332855] davinci_mdio 4a101000.mdio: phy[0]: device 4a101000.mdio:00, driver SMSC LAN8710/LAN8720
[    1.370311] 47401300.usb-phy supply vcc not found, using dummy regulator
[    1.381482] musb-hdrc musb-hdrc.0.auto: Failed to request rx1.
[    1.387819] musb-hdrc musb-hdrc.0.auto: musb_init_controller failed with status -517
[    1.397862] 47401bsb-hdrc musb-hrc.1.auto: mub_init_cotroller faile with staus -517
[    1.439370] i2c /dev entries driver
[    1.445211] pps pps0: new PPS source pps0.-1
[    1.450058] pps pps0: Registered IRQ 75 as PPS source
[    1.456301] pps pps1: new PPS source pps1.-1
[    1.461009] pps pps1: Registered IRQ 50 as PPS source
[    1.467908] omap_hsmmc 48060000.mmc: Got CD GPIO
[    1.548931] ledtrig-cpu: registered to indicate activity on CPUs
[    1.556291] oprofile: using arm/armv7
[    1.561115] Initializing XFRM netlink socket
[    1.565981] NET: Registered protocol family 10
[    1.574051] ip6_tables: (C) 2000-2006 Netfilter Core Team
[    1.579865] sit: IPv6 over IPv4 tunneling driver
[    1.586876] NET: Registered protocol family 17
[    1.591737] NET: Registered protocol family 15
[    1.596465] omap_voltage_late_init[    1.621538] ThumbEE CPU extension supported.
[    1.626079] Registering SWP/SWPB emulation handler
[    1.631206] SmartReflex Class3 initialized
[    1.684523] mmc1: MAN_BKOPS_EN bit is not set
[    1.689919] tps65217 0-0024: TPS65217 ID 0xe version 1.2
[    1.696085] at24 0-0050: 4096 byte 24c32 EEPROM, writable, 1 bytes/write
[    1.703301] omap_i2c 44e0b000.i2c: bus 0 rev0.11 at 400 kHz
[ [    1.723029] mmcblk0: mmc1:0001 Q2J54A 3.64 GiB 
[    1.729270] bq32k: probe of 2-0068 failed with error -5
[    1.735865] omap_i2c 4819c000.i2c: bus 2 rev0.11 at 100 kHz
[    1.756136] musb-hdrc musb-hdrc.1.auto: MUSB HDRC host driver
[    1.762839] musb-hdrc musb-hdrc.1.auto: new USB bus registere[    1.772287] mmcblk0boot0: mmc1:0001 Q2J54A partition 1 2.00 MiB
[    1.779697] mmcblk0boot1: mmc1:0001 Q2J54A partition 2 2.00 MiB
[    1.789669]  mmcblk0: p1 p2 p3 p4
[    1.794420] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002
[    1.801675] usb usb1: New USB device strings: Mfr=3, Product=2, Ser SerialNumber: usb-hdrc.1.uto
[    1.835802] hub 1-0:1.0: USB hub found
[    1.840396] hub 1-0:1.0: 1 port detected
[    1.848537] hctosys: unable to open rtc device (rtc0)
[    1.853881] sr_init: No PMIC hook to init smartreflex
[    1.874806] RAMDISK: Loading 9049KiB [1 disk] into ram disk... done. 0
[    2.377660] VFS: Mounted root (squashfs filesystem) readonly on device 1:0.
[    2.389392] devtmpfs: mounted
[    2.393121] Freeing unused kernel memory: 300K
[    2.845646] devpts: called with bogus options
Running sysctl: OK
Starting rngd: OK
/etc/init.d/S04getrandom: Waiting for sufficient entropy
[    3.670542] random: nonblocking pool is initialized
[    4.791435] EXT4-fs (mmcblk0p4): mounted filesystem with ordered data mode. Opts: (null)
Starting syslogd: OK
Starting klogd: OK
Populating /dev using udev: [    6.106640] udevd[114]: starting version 3.2.9
[    6.602769] udevd[115]: starting eudev-3.2.9
[    7.632520] input: gpio_keys as /devices/platform/gpio_keys/input/input0
[    7.853702] omap_wdt: OMAP Watchdog Timer Rev 0x01: initial timeout 60 sec
[    8.300728] stpm3x: loading out-of-tree module taints kernel.
[    8.310675] stpm3x spi1.0: CRC error reading 0x04
[    8.319975] lm75: probe of 2-0048 failed with error -121
[    8.358070] CAN device driver interface
[    8.420181] leds-pca955x 2-0060: leds-pca955x: Using pca9552 16-bit LED driver at slave address 0x60
[    8.441468] c_can_platform 481cc000.can: c_can_platform device registered (regs=fa1cc000, irq=165)
[    8.453144] c_can_platform 481d0000.can: c_[    8.592889] ina2xx 2-0040: error configuring the device: -121
[    9.294631] Driver for 1-wire Dallas network protocol.
done
Starting network: [   10.570392] Netfilter messages via NETLINK v0.30.
[   10.622530] nf_tables: (c) 2007-2009 Patrick McHardy <kaber@trash.net>
[   11.033735] nf_conntrack version 0.5.0 (5802 buckets, 23208 max)
Waiting for interface eth1 to appear............... timeout!
run-parts: /etc/network/if-pre-up.d/wait_iface: exit status 1
Failed to bring up eth1.
[   27.392084] c_can_platform 481cc000.can can0: setting BTR=1c02 BRPE=0000
[   27.760761] c_can_platform 481d0000.can can1: setting BTR=1c02 BRPE=0000
FAIL
Starting Network Interface Plugging Daemon: eth0.
[   28.011557] net eth0: initializing cpsw version 1.12 (0)
[   28.017184] net eth0: initialized cpsw ale version 1.4
[   28.022670] net eth0: ALE Table size 1024
[   28.029376] net eth0: phy found : id is : 0x7c0f1
Starting ntpd: [   28.154045] IPv6: ADDRCONF(NETDEV_UP): eth0: link is not ready
OK
Starting core file monitoring
Starting cron ... done.
Applying Calibration to iio kernel module
1+0 records in
1+0 records out
2048+0 records in
2048+0 records out
triming data
BusyBox v1.31.1 (2020-07-28 09:15:47 UTC) multi-call binary.

Usage: truncate [-c] -s SIZE FILE...

Truncate FILEs to the given size

	-c	Do not create files
	-s SIZE	Truncate to SIZE

checking data
sha1sum: /tmp/sha1.txt: no checksum lines found
STPM3X config checksum failed, applying default values.
Using unknown device default values
Restarting kernel module with device1_voltage_attenuation_factor=885 device1_current_attenuation_factor=885 device1_power_gain=243270947 device2_voltage_attenuation_factor=885 device2_current_attenuation_factor=885 deoffst=0 devce2_votage_ffset=0devic2_curren_offse=0 deice2_poer_offet=0
[   29.698751] stpm3x spi1.0: CRC error reading 0x04
Starting ssb daemon
[   31.830056] EXT4-fs (mmcblk0p1): mounted filesystem with ordered data mode. Opts: (null)
[   32.159765] EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
[   32.459638] EXT4-fs (mmcblk0p3): mounted filesystem with ordered data mode. Opts: (null)
[   33.316241] EXT4-fs (mmcblk0p2): mounted filesystem with ordered data mode. Opts: (null)
[   33.719782] EXT4-fs (mmcblk0p3): mounted filesystem with ordered data mode. Opts: (null)
```


