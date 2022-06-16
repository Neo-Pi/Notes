###### Uboot
Step1:clone code
```
root@admin:git clone https://github.com/u-boot/u-boot        //源码地址
root@admin:git clone https://gitee.com/mirrors/u-boot         //国内分流
```
Step2:compile code
```
root@admin:d u-boot
root@admin:make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- qemu-riscv64_smode_defconfig
root@admin:make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j$(nproc)
```
###### Open SBI
Step1:clone code
```
root@admin:git clone https://github.com/riscv-software-src/opensbi         //源码地址
root@admin:git clone https://gitee.com/mirrors/OpenSBI                        //国内分流
```
Step2:compile code
```
root@admin:cd opensbi
root@admin:make all ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- PLATFORM_RISCV_XLEN=64 FW_PAYLOAD_PATH="path"/u-boot.bin PLATFORM=generic -j$(nproc)
```
Step3:run test
```
make run PLATFORM=generic CROSS_COMPILE=/opt/riscv64/bin/riscv64-unknown-linux-gnu-
```
```
 AS        platform/generic/firmware/fw_payload.o
 ELF       platform/generic/firmware/fw_payload.elf
 OBJCOPY   platform/generic/firmware/fw_payload.bin
qemu-system-riscv64 -M virt -m 256M -nographic -bios /home/neo/workspaces/opensbi/build/platform/generic/firmware/fw_payload.elf 

OpenSBI v1.0-92-g9dc5ec5
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name             : riscv-virtio,qemu
Platform Features         : medeleg
Platform HART Count       : 1
Platform IPI Device       : aclint-mswi
Platform Timer Device     : aclint-mtimer @ 10000000Hz
Platform Console Device   : uart8250
Platform HSM Device       : ---
Platform Reboot Device    : sifive_test
Platform Shutdown Device  : sifive_test
Firmware Base             : 0x80000000
Firmware Size             : 288 KB
Runtime SBI Version       : 0.3

Domain0 Name              : root
Domain0 Boot HART         : 0
Domain0 HARTs             : 0*
Domain0 Region00          : 0x0000000002000000-0x000000000200ffff (I)
Domain0 Region01          : 0x0000000080000000-0x000000008007ffff ()
Domain0 Region02          : 0x0000000000000000-0xffffffffffffffff (R,W,X)
Domain0 Next Address      : 0x0000000080200000
Domain0 Next Arg1         : 0x0000000082200000
Domain0 Next Mode         : S-mode
Domain0 SysReset          : yes

Boot HART ID              : 0
Boot HART Domain          : root
Boot HART Priv Version    : v1.10
Boot HART Base ISA        : rv64imafdc
Boot HART ISA Extensions  : time
Boot HART PMP Count       : 16
Boot HART PMP Granularity : 4
Boot HART PMP Address Bits: 54
Boot HART MHPM Count      : 0
Boot HART MIDELEG         : 0x0000000000000222
Boot HART MEDELEG         : 0x000000000000b109

Test payload running
```
###### Copy the opensbi binary file to SD card
Step1:insert  SD card->Partition the SD card
```
root@admin:fdisk -l
```
```
Disk /dev/sdb: 28.89 GiB, 31021072384 bytes, 60588032 sectors
Disk model: Storage Device  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: FA5DD2AB-FAA9-4F89-B510-3B4303B06E69

Device     Start      End  Sectors  Size Type
/dev/sdb1   2048    67583    65536   32M ONIE boot
/dev/sdb2  67584 60587998 60520415 28.9G Linux filesystem
```
```
root@admin:sgdisk --clear --new 1:0:4Mib --new 2 --typecode=1:3000 --typecode=2:8300 /dev/sdb
```
```
The operation has completed successfully.
```
Step2:if you partition the SD card fail,you can try:
```
root@admin:parted /dev/sdb
```
```
GNU Parted 3.4
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklable gpt
```

```
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No?  yes
```

```
(parted) quit
```
Then try Step1

Step3:open the opensbi fire
```
root@admin:cd opensbi/build/platform/generic/firmware/
root@admin:ls
```
```
fw_dynamic.bin  fw_dynamic.elf     fw_dynamic.o  fw_jump.dep  fw_jump.elf.ld  fw_payload.bin  fw_payload.elf     fw_payload.o
fw_dynamic.dep  fw_dynamic.elf.ld  fw_jump.bin   fw_jump.elf  fw_jump.o       fw_payload.dep  fw_payload.elf.ld  payloads
```
Step4:copy the fw_payload.bin to SD card
```
dd if=fw_payload.bin of=/dev/sdb1 status=progress oflag=sync bs=1M
```
Insert the SD card in FPGA and press reboot key.
