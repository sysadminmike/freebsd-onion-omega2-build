Onion Omega2+ FreeBSD build

See https://github.com/freebsd/freebsd-wifi-build/wiki for reference.

    % cat ~/.freebsd-wifi-build-settings.cfg 
    X_SKIP_MORE_STUFF=YES
    X_EXTRA_SRC_CONF=~/omega2/src.conf

    % cat ~/omega2/src.conf
    WITHOUT_CLANG=YES
    WITHOUT_CLANG_FULL=YES

get dts files from thread http://community.onion.io/topic/1099/openwrt-on-the-omega-2/8


    cd ~/omega2/src/sys/gnu/dts/mips
    fetch https://raw.githubusercontent.com/WereCatf/source/image/target/linux/ramips/dts/OMEGA2.dtsi
    fetch https://raw.githubusercontent.com/WereCatf/source/image/target/linux/ramips/dts/OMEGA2.dts
    fetch https://raw.githubusercontent.com/WereCatf/source/image/target/linux/ramips/dts/OMEGA2P.dts


Edit src/sys/mips/mediatek/std.mediatek and comment out debugging stuff as i had a problem with them enabled:

    # Debugging for use in -current
    #options         INVARIANTS
    #options         INVARIANT_SUPPORT
    #options         WITNESS
    #options         WITNESS_SKIPSPIN
    #options         DEBUG_REDZONE
    #options         DEBUG_MEMGUARD



To run build need to specify DTS file so in bash/sh:

    X_DTS_FILE=OMEGA2P.dts KERNCONF=MT7628_FDT ../freebsd-wifi-build/build/bin/build ralink buildworld
    X_DTS_FILE=OMEGA2P.dts KERNCONF=MT7628_FDT ../freebsd-wifi-build/build/bin/build ralink buildkernel
    X_DTS_FILE=OMEGA2P.dts KERNCONF=MT7628_FDT ../freebsd-wifi-build/build/bin/build ralink installworld
    X_DTS_FILE=OMEGA2P.dts KERNCONF=MT7628_FDT ../freebsd-wifi-build/build/bin/build ralink installkernel
    X_DTS_FILE=OMEGA2P.dts KERNCONF=MT7628_FDT ../freebsd-wifi-build/build/bin/build ralink distribution
 
For Omega2 owners user X_DTS_FILE=OMEGA2.dts instead.  
 
You should now have a kernel in ../tfpboot

Covert to binary:

    objcopy -O binary kernel.MT7628_FDT kernel.MT7628_FDT.kbin

Compress:

    /usr/local/bin/lzma e kernel.MT7628_FDT.kbin kernel.MT7628_FDT.kbin.lzma

Make a u-boot image:

    mkimage -A mips -O linux -T kernel -C lzma -a 0x80001100 -e 0x80001100 -n "FreeBSD" -d kernel.MT7628_FDT.kbin.lzma kernel.MT7628_FDT.lzma.uImage

Note: If this does not boot check the address of the kernel entry point with 

    readelf -h tftpboot/kernel.MT7628_FDT 

Copy to a usb fat formatted usb key:

    # mount_msdosfs /dev/da0 /mnt
    # cp kernel.MT7628_FDT.lzma.uImage /mnt
    # umount /mnt 


Plug into Omega in expansion dock and turn on while holding the reset button with console open.  

Press 1 to get into the command line.

You will need to reset the usb:

    Omega2 # usb reset

Then load the kernel boot image:

    Omega2 # fatload usb 0:1 0x80800000 kernel.MT7628_FDT.lzma.uImage

And boot it:

    Omega2 # bootm 0x80800000 
    
    
At this point FreeBSD should boot and 


Below is what should happen:

       ____       _             ____
      / __ \___  (_)__  ___    / __ \__ _  ___ ___ ____ _
     / /_/ / _ \/ / _ \/ _ \  / /_/ /  ' \/ -_) _ `/ _ `/
     \____/_//_/_/\___/_//_/  \____/_/_/_/\__/\_, /\_,_/
     W H A T  W I L L  Y O U  I N V E N T ? /___/"
    
    Board: Onion Omega2 APSoC DRAM:  128 MB
    relocate_code Pointer at: 87f60000
    flash manufacture id: c2, device id 20 19
    find flash: MX25L25635E
    *** Warning - bad CRC, using default environment
    
    ============================================
    Onion Omega2 UBoot Version: 4.3.0.3
    --------------------------------------------
    ASIC 7628_MP (Port5<->None)
    DRAM component: 1024 Mbits DDR, width 16
    DRAM bus: 16 bit
    Total memory: 128 MBytes
    Flash component: SPI Flash
    Date:Oct 18 2016  Time:17:29:05
    ============================================
    icache: sets:512, ways:4, linesz:32 ,total:65536
    dcache: sets:256, ways:4, linesz:32 ,total:32768
    CPU freq = 575 MHZ
    Estimated memory size = 128 Mbytes
    Resetting MT7628 PHY.
    Initializing MT7688 GPIO system.

    
    **************************************
    * Hold Reset button for more options *
    **************************************
    
    
    You have 40 seconds left to select a menu option...
    
    
    Please select option:
       [ Enter ]: Boot Omega2.
       [ 0 ]: Start Web recovery mode.
       [ 1 ]: Start command line mode.
       [ 2 ]: Flash firmware from USB storage.

    Option [1] selected.
    
    1: System Enter Boot Command Line Interface.
    
    U-Boot 1.1.3 (Oct 18 2016 - 17:29:05)
    Omega2 # usb reset
    (Re)start USB...
    LOW LEVEL INIT USB0:
    Scanning bus 0 for devices...
    New Device 0
    ...
    value 0x302 index 0x409 length 0xFF
    usb_control_msg: status = success?
    Manufacturer
    Product      USB Flash Drive
    SerialNumber 070B00012340350
    Device is a hub?
    2 USB Device(s) found
    scan end
           Scanning bus for storage devices...
    
    =================================================
    1: Hub,  USB Revision 1.10
     -  OHCI Root Hub
     - Class: Hub
     - PacketSize: 8  Configurations: 1
     - Vendor: 0x0000  Product 0x0000 Version 0.0
    
    =================================================
    2: Mass Storage,  USB Revision 2.0
     -  USB Flash Drive 070B00012340350
     - Class: (from Interface) Mass Storage
     - PacketSize: 64  Configurations: 1
     - Vendor: 0x1005  Product 0xb113 Version 1.0
    Testing BULK mode...Identifying a storage device...*
    USB_STORAGE: 1 Storage Device(s) found
    

    Omega2 #  fatload usb 0:1 0x80800000 kernel.MT7628_FDT.lzma.uImage
    *
    *
    Reading file "kernel.MT7628_FDT.lzma.uImage"
    *
    **
    ******
    **************************************************************************************************************************
    ************
    *
    FAT: 1144975 Bytes read
    Omega2 # bootm 0x80800000
    ## Booting image at 80800000 ...
       Image Name:   FreeBSD
       Image Type:   MIPS Linux Kernel Image (lzma compressed)
       Data Size:    1144911 Bytes =  1.1 MB
       Load Address: 80001100
       Entry Point:  80001100
       Verifying Checksum ... OK
       Uncompressing Kernel Image ... OK
    No initrd
    ## Transferring control to Linux (at address 80001100) ...
    ## Giving linux memsize in MB, 128

    Starting kernel ...

    FDT DTB  at: 0x804133c0
    CPU   clock:  580MHz
    Timer clock:  290MHz
    UART  clock:   40MHz

    U-Boot args (from 0 args):
            None
    Environment:
            memsize=128
            initrd_start=0x00000000
            initrd_size=0x0
            flash_start=0x00000000
            flash_size=0x2000000
    entry: mips_init()
    RAM size: 128MB (from FDT)
    Cache info:
      picache_stride    = 4096
      picache_loopcount = 16
      pdcache_stride    = 4096
      pdcache_loopcount = 8
      max line size     = 32
      cpu0: MIPS Technologies processor v85.150
      MMU: Standard TLB, 32 entries (4K 16K 64K 256K 1M 16M 64M 256M pg sizes)
      L1 i-cache: 4 ways of 512 sets, 32 bytes per line
      L1 d-cache: 4 ways of 256 sets, 32 bytes per line
      L2 cache: disabled
      Config1=0xbee3519e<PerfCount,WatchRegs,MIPS16,EJTAG>
      Config2=0x80000000
      Config3=0x2420<ULRI>
      Config7=0x80010400<WII,AR>
    Physical memory chunk(s):
    0x47b000 - 0x7ffffff, 129519616 bytes (31621 pages)
    Maxmem is 0x8000000
    KDB: debugger backends: ddb
    KDB: current backend: ddb
    Copyright (c) 1992-2017 The FreeBSD Project.
    Copyright (c) 1979, 1980, 1983, 1986, 1988, 1989, 1991, 1992, 1993, 1994
            The Regents of the University of California. All rights reserved.
    FreeBSD is a registered trademark of The FreeBSD Foundation.
    FreeBSD 12.0-CURRENT #0 r317887M: Wed May 10 20:49:10 UTC 2017
        mike@f64-current.mw.office:/usr/home/mike/omega2/obj/mipsel_ap/mips.mipsel/usr/home/mike/omega2/src/sys/MT7628_FDT mips
    gcc version 4.2.1 20070831 patched [FreeBSD]
    Preloaded elf kernel "kernel" at 0x8046e160.
    real memory  = 134217728 (131072K bytes)
    Physical memory chunk(s):
    0x0050f000 - 0x07d9ffff, 126423040 bytes (30865 pages)
    avail memory = 125616128 (119MB)
    




For mfsroot:


    X_DTS_FILE=OMEGA2P.dts KERNCONF=MT7628_FDT ../freebsd-wifi-build/build/bin/build ralink mfsroot
    X_DTS_FILE=OMEGA2P.dts KERNCONF=MT7628_FDT ../freebsd-wifi-build/build/bin/build ralink fsimage

You should now have a populated mfsroot folder.

Copy the content of this to another usb memory stick - or partition the first with msdos for kernel part and ufs for mfsroot.

    gpart destroy -F da0
    gpart create -s MBR da0
    gpart add -t freebsd da0
    newfs /dev/da0s1
    mount /dev/da0s1 /mnt
    cpdup mfsroot/ralink/ /mnt
    chown -R root:wheel /mnt
    umount /mnt
    
    
Follow above instructions to boot from kernel on fat formatted usb memory stick, but before running "bootm 0x80800000" remove the kernel usb key and replace it with the mfs root one.
At the mountroot prompt enter the ufs usb key:

    mountroot> ufs:/dev/da0s1

And you should see something like the below:


    da0: Delete methods: <NONE(*),ZERO>
    Mounting from ufs:md0.uzip failed with error 19.
    
    Loader variables:
    
    Manual root filesystem specification:
      <fstype>:<device> [options]
          Mount <device> using filesystem <fstype>
          and with the specified (optional) option list.
    
        eg. ufs:/dev/da0s1a
            zfs:tank
            cd9660:/dev/cd0 ro
              (which is equivalent to: mount -t cd9660 -o ro /dev/cd0 /)
         
      ?               List valid disk boot devices
      .               Yield 1 second (for background tasks)
      <empty line>    Abort manual input
    
    mountroot> ufs:/dev/da0s1
    Trying to mount root from ufs:/dev/da0s1 []...
    warning: no time-of-day clock registered, system time will not be set accurately
    start_init: trying /sbin/init
    May 11 22:07:41 init: login_getclass: unknown class 'daemon'
    *** Mounting /tmp, /var, /etc ...
    random: unblocking device.
    *** Copying /c/etc -> /etc ...
    *** Starting rc2 ...
    *** Populating /var ..
    *** Loading configuration files ..
    *** Restoring from  ..
    dd: no value specified for if
    gunzip: (stdin): unexpected end of file
    0 blocks
    *** Completed.
    *** setting up hostname
    *** Load kernel modules
      .. bridgestp
    kldload: can't load bridgestp: module already loaded or in kernel
      .. if_bridge
    kldload: can't load if_bridge: No such file or directory
      .. random
    kldload: can't load random: No such file or directory
    *** Setting kern.random.harvest.mask=511
    kern.random.harvest.mask: 2047 -> 511
    *** bringing up loopback ..
    *** Default password/login databases ..
    *** Customising sysctl ..
    *** Starting networking via /etc/rc.d/base/net
    *** Interface: rt0: start
    *** Interface: rt0: done
    *** Interface: bridge0: start
    bridge0: bpf attached
    bridge0: Ethernet address: 06:a3:b1:0e:30:1d
    rt0: entering promiscuous mode
    rt0: promiscuous mode enabled
    bridge0: link state changed to UP
    *** Interface: bridge0: done
    ipfw: setsockopt(IP_FW_XDEL): Protocol not available
    *** Done!
    
    FreeBSD/mips (freebsd-wifi) (ttyu0)
    
    login:

