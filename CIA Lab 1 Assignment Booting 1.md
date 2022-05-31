###### tags: `CIA`
:::success
# CIA Lab 1 Assignment: Booting 1
**Name: Radomskiy Ilia**
:::




## Task 1: PXE Installation
:::info
PXE Server Setup
:::



## Implementation:

The tirst step is to set up two virtual machines:
1) PXE Server Ubuntu [1]
2) Ubuntu Client
We will be using it in order to check PXE Server working (DHCP, TFTP included)



<center>

![](https://i.imgur.com/Y57ekUl.png)

Figure 1: Virtual box manager with two working machines
</center>

In the below picture you can see working DHCP and TFTP service on the "PXE Server Ubuntu" machine

<center>

![](https://i.imgur.com/rSeDeFd.png)

Figure 2: DHCP and TFTP service
</center>

Folders in tftp - /var/lib/tftpboot

In order to not interfere with the SNE network, we will be using isolated virtual machines. You can simply do that by setting up "Internal network" in network settings as shows in figure 3. Interference of several DCHP services may lead to network issues


<center>

![](https://i.imgur.com/12amq0D.png)

Figure 3: Isolating network from the SNE network
</center>

In order to or DHCP to work we will configure some of network information: DHCP range, subnet, routers, broadcast, and time lease
```
range 192.168.1.150 192.168.1.200;
option subnet-mask 255.255.255.0;
option routers 192.168.1.1;
option broadcast-address 192.168.1.255
default-lease-time 600;
max-lease-time 7200;
```
we also need to configure DHCP setting in order to have ability to load from network on other machines
Settings are listed below, and displayed on figure 4
In this we tell our server what is an IP address of PXE server and file name of OS
```
allow booting;
allow bootp;
option option-128 code 128 = string;
option option-129 code 129 = text;
next-server 192.168.1.100
filename "pxelinux.0"
```

<center>

![](https://i.imgur.com/FbSMP3I.png)
![](https://i.imgur.com/PyoRqj9.png)
![](https://i.imgur.com/EBwZBOf.png)

Figure 4: DHCP configuration
</center>

Ramdisk-size - tells to the kernel about need in creation ram disk in the memory. Use main memory as a block device

Block device - something used for discrabing how does it is requared to interact with the rosourses (physical). Somethig  witk cloud computing

kernel - main progtramm of the os, gives application access to the resources
<center>

![](https://i.imgur.com/KH81YCT.png)
![](https://i.imgur.com/o6wUdYo.png)

Figure 5: Tftp configuration
</center>

:::info
Ubuntu clients
:::

In order to check our DHCP server we login in "Ubuntu client" and receive DHCP address

<center>

![](https://i.imgur.com/zigHvAV.png)
Figure 5: DHCP client gets IP 192.168.1.150
</center>

From this point, we are ready to send OS from our PXE server to a new client
To this end, we create a new Virtual Machine, which can be seen in figure 1
In figure 6.1 you can see the process of acceptance of IP address and OS image for newly created machine

Boot order like from f12 then we are choosing network

uefi or bios
bios - basic in out system
uefi - extensible firmvafe interface

check ls /sys/firmware/efi
or
sudo apt install efibootmgr
sudo efibootmgr

<center>

![](https://i.imgur.com/HOuYWmb.png)

Figure 6.1: Second DHCP client

![](https://i.imgur.com/dydKCK1.png)

Figure 6.2: Second DHCP client
</center>

vnet is used for connection of different virtual bridges
virbr vs vnet
virbr - briedge - no connection
vnet - tap - conected to nic

**  **

uefi - in extantion to the bios

secure boot - not allowing to execute cone with no signature remotely

---
## Task 2: Questions to answer

:::info 
1a) What is UEFI PXE booting?
:::
UEFI (Extensible Firmware Interface) - the main purpose is to initialize interfaces when our system is booting. Some kind of updated version of BIOS

PXE (Preboot Execution Environment) - set of protocols (BOOTP/DHCP/TFTP) that allows manipulating PCs of clients in the network, for example, install OS or other programs

UEFI PXE - Have similar differences to PXE, as UEFI and BIOS. UEFI PXE is just an updated version of PXE, so instead of BIOS in the older version UEFI will be used [2]

:::info 
1b) How does it work?
:::

UEFI PXE consist of TFTP, which is used for downloading files from the server, DHCP, this protocol  is used for automatic IP assignment
1 when the new device boots up it sends a broadcast DHCP request
2 If there is a DHCP server in this network it sends a suggestion to the new device
2.1 If there is a PXE server in this network with an IP suggestion server send information about the PXE server
3 If a new device needs to load from PXE it looks for possibilities  and requests OS, like in figure 6.1

:::info 
2a) What is a GPT?
:::
GPT (GUID Partition Table) - A better version of MBR (because of new capacity  limit) used to "mark up" parts of the disk

:::info 
2b) What is its layout? Explain each element.
:::
LBA 0 - used for compatibility with old software also protect the GPT structure from accidental damage during the operation of programs. Similar to MBR

LBA1 - contain data about every LBA address, that is used for partitioning

LBA2 - Disk partition table

LBA34 - Data of disk partitions

LBA-34 - a copy of LBA2

LBA-2 - Copy of GPT header 

LBA-1 - We use it when the first element (LBA 0) is corrupt[5]

<center>

![](https://i.imgur.com/iIprD03.png)
Figure 7: GPT layout
</center>

:::info 
2c) What is the role of a partition table?
:::
We need disk partition in order to manage storage separately. The partition table is some kind of map, that describes what partitions are on disk

:::info 
3a) What is gdisk?
:::

gdisk - Interactive GUID partition table (GPT) manipulator. It is used to create and changing of partition tables[6] 

:::info 
3b) How does it work?
:::

Start: At the beginning gdisk trying to identify the type of disk. If the disk can use GPT, it will use it. But if there is no GPT, only MBR, gdisk will try to convert it to GPT

:::info 
3c) What can you do with it?
:::

The program allows you to do several things with partition tables. For example
Save data about partitions to file, you can read it by using ```hd``` or ```xxd```
Delite partitions
Create new partitions
Show information about partitions
And write data to save changes


:::info 
4a) What is a Protective MBR and why is it in the GPT?
:::

Protective MBR is contained in LBA0 of GPT and used to 'protect' GPT from old systems. Old systems only can work with MBR.

## Task 3: Partitions
:::info
1 Copy and dump the Protective MBR and GPT in hex format
:::
In order to create a dump of MBR and GPT, we will be using dd utility
```
dd if=/dev/sda of=mbr.txt bs=512 count=1
dd if=/dev/sda of=GPT_TABLE.txt bs=1 count=1536
```
You may see the result of the dump in hex format using hd utility. For MBR in figure 8, and for GPT in figure 9

<center>

![](https://i.imgur.com/HXb60dj.png)
Figure 8: MBR dump
</center>


The first part of GPT is actually MBR, this part is used if an old device is trying to load. So both of both dumps contain pretty much the same lines, ends up with 55AA, this is the END OF MBR marker or boot signature. Let's take a closer look at the MBR dump.

The first line of High lighted lines is 
```
0000 0200 eeff ffff 0100 0000 2f60 383a
```
In the table below, you may see from what parts does this line consists of [7]

| 00 | 00200 | ee | ffffff | 0100 0000 | 2f60 383a |
| -- | ----- | -- | ------ | --------- | --------- |
|Status or physical drive|Address of the first absolute sector in partition(in CHS)|Partition Type|Last Sector(in CHS)|Relative Sectors(or Offset)|Total Sectors(or Length)|
|may be 80 or 00|Location of GPT|'ee' equals to EFI_GPT_DISK|16777215|16|976773167 or 465 GiB|


<center>

![](https://i.imgur.com/hsEYIVh.png)

Figure 9: GPT dump
</center>


```
0000 0200 eeff ffff 0100 0000 2f60 383a
```

In the table below, you may see the GPT header and its hex dump


| 45 46 49 20 | 50 41 52 54 | 00 00 01 00  |  5c 00 00 00 | 
| ------------|----------- | ------------ | ------------ |
| ASCII: "EFI PART" identificator | ASCII: "EFI PART" identificator | Revision number for this header| Header Size|
|||Curent GPT Header versoion is still 1| must be more or equal 92 (our case), or less or equal to logical block size|
| 12 b0 13 f9 | 00 00 00 00 | 01 00 00 00  |  00 00 00 00 | 
|CRC32 of the GPT Header|Reserved; must be set to zero | LBA location of this GPT Header|LBA location of this GPT Header|
|||most oftenly equals to |0x0000000000000001|
| ff ff df 01 | 00 00 00 00  | 22 00 00 00  |  00 00 00 00 | 
|LBA address of the alternate GPT Header, back up copy  |LBA address of the alternate GPT Header, back up copy |First usable LBA Sector for any partition |First usable LBA Sector for any partition|
|This valie points to the sector|31457279|Most often Sector 34| 22 hex = 34 dec
| de ff df 01 | 00 00 00 00  | 0b 9e 70 af  |  ad b5 dd 49 | 
|Last usable LBA Sector for any partitions|Last usable LBA Sector for any partitions|Disk GUID |Disk GUID
|In our case last sector is |4194254||
| 8e a0 14 d0 | a3 b5 24 a1  | 02 00 00 00  |  00 00 00 00 | 
|Disk GUID|Disk GUID|Starting LBA of the array of GUID Partition entries |Starting LBA of the array of GUID Partition entries
||||
| 80 00 00 00 | 80 00 00 00  | 6c 98 27 0e  |  00 00 00 00 | 
|Number of Partition Entries in the GUID Partition Entry array  |Size, in bytes, of each GUID Partition Entry |CRC32 of the GUID Partition Array |block reserved by UEFI
|number of partitions is 128|Partition Entry is 128||

CRC32 means Cyclic redundancy check, its is an algorithm of data integrity checks

Values should be readen in little-endian, so the Disk GUID will be
```
0Xa124b5a3d014a08e49ddb5adaf709e0b
```
In the table below, you may see the GPT header

<center>

![](https://i.imgur.com/tuZ7lYn.png)
Figure 10: GPT header
</center>

On the next figure, you may see partition table array entries of the GPT table

<center>

![](https://i.imgur.com/xMIi7cS.png)
Figure 11: Table array entries of the GPT
</center>

Let's take a closer look at it in the table below

| 28 73 2a c1 1f f8 d2 11 | ba 4b 00 a0 c9 3e c9 3b| 
| ------------------------|----------------------- |
| Unique ID that defines the purpose and type of this Partition|Unique ID that defines the purpose and type of this Partition|
|||
| ec e4 91 95 fb ff b0 4c | 9a 2e 55 9e 35 35 7a 38|
|GUID that is unique for every partition entry|GUID that is unique for every partition entry|
|||
| 00 08 00 00 00 00 00 00 | ff 07 10 00 00 00 00 00|
|Starting LBA|Ending LBA|
|||
| 00 00 00 00 00 00 00 00 | 45 00 46 00 49 00 20 00|
|Reserved (normally all zero bytes)|Partition Name, human-readable|
| 53 00 79 00 73 00 74 00 | 65 00 6d 00 20 00 50 00|
|Partition Name, human-readable|Partition Name, human-readable|
| 61 00 72 00 74 00 69 00 | 74 00 69 00 6f 00 6e 00|
|Partition Name, human-readable|Partition Name, human-readable|



:::info
a) At what byte index from the start of the disk do the real partition table entries start?
:::
Because on my disk Entries start from sector 3 (two is  already exist). One sector is 512 logical sectors, but the start of the record is 0x400 or 1024 in dec

:::info
b) At what byte index would the partition table start if your server had a so-called “4K native”
(4Kn) disk?
:::
4K native is a technology that allows maintaining 8 sectors x512 bytes = 4096 logical sectors. It allows improving efficiency. And start byte index will be 0x2000 hex = 8192 dec

:::info
2 Add partition called ØS3 to the table by hand
What values would you have to use for the entry?
:::

To add a new FreeBSD ZFS partition by hand we should change some of the parts а GPT. The main thing to add is the GUID of FreeBSD ZFS partition
```
516E7CBA-6ECF-11D6-8FF8-00022D09712B
```
But, the actual code phrase in the table will be like:
```
BA7C6E51-CF6E-D611-8FF8-00022D09712B
```
Instead of
```
45464920 5041 5254 0000 0100 5c000000
```
So it is small endian for the first three blocks

Adding specific name to the partition is also consideration. So instead of current name 
```
0045 0046 0049
```
We will write
```
00D8 015A 0033
```
In the GBT partition table on the figure 12
<center>

![](https://i.imgur.com/uxQcyTO.png)
![](https://i.imgur.com/6BjfSR4.png)

Figure 12: GBT partition table
</center>
torcher
:::info
3 Differences between primary and logical partitions in an MBR partitioning scheme.
:::

Primary partitions - only may by 4 of them, or 3 and 1 extended, the extended  one may contain a lot of logical partitions: 23
Logical partitions - also called LPAR, main difference with primary partitions is that you can't start  OS from logical partition

---
## References:

1: [How to install PXE Server on Ubuntu](https://ostechnix.com/how-to-install-pxe-server-on-ubuntu-16-04/)
2: [Preboot Execution Environment (PXE) Wikipedia](https://en.wikipedia.org/wiki/Preboot_Execution_Environment)
3: [How does the PXE boot process work?](https://techcommunity.microsoft.com/t5/itops-talk-blog/how-does-the-pxe-boot-process-work/ba-p/888557)
4: [GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table)
5: [Изучаем структуры MBR и GPT](https://habr.com/ru/post/347002/)
6: [Man page of GDISK](https://www.rodsbooks.com/gdisk/gdisk.html)
7: [MBR/EBR Partition Tables](https://thestarman.pcministry.com/asm/mbr/PartTables3.htm)






