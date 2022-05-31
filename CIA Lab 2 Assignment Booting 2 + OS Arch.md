###### tags: `CIA`
:::success
# CIA Lab 2 Assignment: Booting 2 + OS Arch 
**Name: Radomskiy Ilia**
:::

---
## Task 1: Booting the OS
:::info
1 What is a UEFI OS loader and where does the Ubuntu OS loader reside on the system?
:::
UEFI (The Unified Extensible Firmware Interface) is the specification that describes communication methods between the operating system and firmware.
It can be stored in memory MBR, GPT, USB or in the network
stored in 

![](https://i.imgur.com/z9VFp6Z.png)

[1]

:::info
2 Describe in order all the steps required for booting the computer (until the OS loader starts running.) Ubuntu uses the UEFI OS loader to load and run the GRUB boot loader
:::

Boot stages:
1 SEC (Security Phase) - the first stage of the UEFI boot loader. This is assembly code describes the architecture and initializes CPU cache as a memory. It also Launches the next stage booting - PEI and tells to refer to it as a root, so trust it.

2 PEI (Pre-EFI Initialization) - Because at this stage RAM isn't accessible, the CPU uses cache as RAM. This stage loads up dependencies between main memory and firmware.

3 DXE (Driver Execution Environment) - This is the first step of the device initialisation process, on this stage available CPU, chipset, mainboard and peripherals are initialized in DXE and BDS.

4 BDS (Boot Device Select) - On the second stage of the Device initialisation process PCI devices are executed according to configuration

5 TSL (Transient System Load) - On this stage Human can actually interact with the UEFI OS loader or GRUB, you may use the UEFI OS loader shell. From this point, you could choose OS via OS boot loader

6 RT (Runtime) - On this stage OS loader gives control to OS, ```Ubuntu``` in our case [2]
 

:::info
3 What is the purpose of the GRUB boot loader in a UEFI system?
:::

GRUB (Grand Unified Bootloader) this program provides multiple boot options for the user in the booting process, the choice is provided before loading the kernel of the OS. We need GRUB simply because MBR does not have enough space to contain a multi-OS  boot loader, but enough to load GRUB from another space from the hard disk [3]

Also UEFI does not know kernel file type so grub helps it
and responsible for loading and tranfering control to the operation system kernel

:::info
4 How does the Ubuntu OS loader load the GRUB boot loader?
:::

Usually, GRUB initializes OS and may give to the user several options of booting. But user can tell the GRUB what OS is controlling the boot loader. In this case, this OS is controlling the boot menu
[4]

shimx64 program that provides a way to boot on a computer with Secure Boot active, not allowin to start programs without signature, then starts grubx64

grubx64 - grub binary

:::info
5 Explain how the GRUB boot loader, in turn
:::

kernel is on /boot/

ext (extended file system) first file system for the linux. if there is number that means version

vmlinuz - linux kernel executable

initrd - initial ram disk temporary file system used by linux in first stage of booting

a) What type of filesystem is the kernel on?

We can see what filesystem is supported by currently running kernel [5]
```
cat /proc/filesystems
```
In our case

```
nodev	sysfs
nodev	tmpfs
nodev	bdev
nodev	proc
nodev	cgroup
nodev	cgroup2
nodev	cpuset
and others
```

b) What type(s) of filesystem does UEFI support?

UEFI expects that applications are stored in a FAT12, FAT16 or FAT32 file system on a GPT. But it is more often to used FAT32 by firmwares[6]

c) What does the GRUB boot loader, therefore, have to do to load the kernel?

GRUB that is installed in the directory ```'/dev/sda'```, our hard disk, needs to know about OS on the hard drive. That is why we might need to specify it

```
grub-mkrescue -o myos.iso isodir
```

This command locates OS's in the ```'isodir'``` directory, so when it is needed the config GRUB file will know about them. File 'myos.iso' is an image of the OS.[7]

:::info
6 Do you need an OS loader and/or boot loader to load a Linux kernel with UEFI? Explain why or why not.
:::

No. the Linux kernel can be executed directly by UEFI. To this end ```'EFISTUB'``` may be used. This option might be chosen if user don't have multiple OS boot options [8]

:::info
7 In an MBR-based system, the GRUB bootloader is distributed over the disk in multiple parts.
:::

a) How many parts (or stages) does GRUB have in such a system, and what is their task?

Because MBR does have not enough space for the full BRUB boot loader, the booting process is taken several stages
Stage 1: Written in MBR, it leads to the next GRUB stage 1,5 or 2
Stage 1,5: On this stage, the user may specify the type of the file stages
Stage 2: This is the main image of GRUB, it reads grub configuration, and funds out the way of kernel booting. It supports GUI 

b) Where are the different stages found on the disk?

You can find Stages of GRUB in the next locations:
Stage 1: First 446 bytes of MBR
Stage 1.5: In the folder ```/boot/grub``` we can see configuration files for the file systems
Stage 2: In this stage ```/boot/grub/grub.conf``` is readed in order to get information about configuration [9]

---
## Task 2: Initializing the OS

:::info
1 Describe the entire startup process of Ubuntu in the default installation.
:::

a) What is the first process started by the kernel?

The bootloader starts the kernel running. 
After that kernel starts the ```'initrd'``` process. 
```'initrd'``` stands for (Initial RAM Disk) this is a temporary file system, that is used by the Linux kernel. Usually, it is used for the first initialization of OS. The kernel starts the script of ```'initrd'``` inside the file system, which loads hardware drivers and finds the root partition. [10]
after the kernel is loaded into memory and begins working, it extracting itself, after that process kernel executes the ```'init'``` process. "Init" is short for "initialization" so to say it initialize different scripts that set up ready to run OS. [11]
But now process ```'systemd'``` replaced ```'init'```. It has the same responsibilities: to start the process, but it makes it quicker.

b) Where is the configuration kept for the started process?

You may find kernel configuration by printing  
```
less /boot/config-*
```
where * is a version of your kernel

```'init'``` process is located in the directory ``` /sbin/init```

```'systemd'``` is stored in ```/etc/systemd/system/```

But for compatibility with the old systems ```'init'```  launches    [12]

c) It starts multiple processes. How is the order of execution defined?

On this point, when ```'init'``` process is running, it executes all scripts for the default run level
At this stage, a log of scripts runs sequentially
After everything is set up ```init``` spawn ```getty``` programs, this program prompts for login of the user, user authentification is checked by programm ```login```
User login stored in ```/bin/login```
If authentification  is successful ```login``` starts the shell

In order to improve the speed of booting, ```'dependencies'``` were implemented as a part of ```systemd```, let's say one service (sshd.service) needs to be started after another (network.target). If (network.target) second service ID is not ready then the first one (sshd.service) is waiting. The process itself might be imagined like a tree with the ```'systemd'``` process as a root.
In ```'systemd'``` case analogy  with the tree is no accurate because some of the processes cannot do anything, without every other process started before, this creates some kind of bottleneck that is shown in figure 1. [13]

<center>

![](https://i.imgur.com/2uaktGB.png)

Figure 1: Boot up logic of the ```'systemd'```

</center>


---
## Task 3: Basic OS interaction

:::info
 Binaries and scripts
:::

Lits find some information about binaries from the directory ```/usr/bin``` via ```file``` command
In figure 2, you can see the output of the script
The basic output of the file is:
directory of the file: class of the file, data, version of the OS, likability, directory of interpreter, version of the build, OS version

<center>

![](https://i.imgur.com/nuV9iOk.png)
Figure 2: ```file``` output
</center>


:::info
 Tracing binaries
:::

1) Use “strace” to find what other systems call besides “stat” “zsh” uses before executing an “execve” system call to start a given binary

Besides information about the ELF file we can track the process of execution by using command ```'strace'```

The output of the command ```strace zsh -c /bin/ls``` is too big to show it on the figure. But we can describe some of the systems calls in it. They will be described in order of execution.

execve - this system call executes a program that printed in the command, in our case ```'ls'```
brk - change address location of the data segment, creates new ending in the memory
arch_prctl - trying to set specific architecture or tread
access - this system call checks if the directory is reachable
openat - open up a file descriptor
fstat - writes information to the buffer, pointer - ```statbuf```
mmap - writes the common file to the process memory
Close - closes the file descriptor, from this point descriptor can be recalled
read - this system call tries to read count bytes from the descriptor in the buffer
pread64 - from the file descriptor amount of count bytes is read into offset, on the kernel version 2.6 this call is named ```pread64``` instead of ```pread```
mprotect - is one of the protection calls, this system call deny illegal access to the memory slot that the process does not have access to
prctl - is the multifunctional process, in order to specify ist action we use different first arguments (e.g. PR_CAPBSET_READ for reading)
ioctl - manipulates parameters of the device or files
readlink - creates a link to the path in the buffer
dup - duplicates file descriptor, this descriptor has a link to the same file
getppid - gives ID of the calling process
getuid - gives user ID
socket - returns file descriptor by creating an endpoint
connect - creates a connection to the socket
lseek - sets up offset for the read or write operations
uname - kernel information
prlimit64 - this system call creates a resource limit for the process
stat - file status
fcntl - allow operating file descriptor
rt_sigaction - this call may change the current action of the process or check them
getrusage - resourse ussadge
rt_sigpromask - allows setting mask for currently blocked (time reserved) thread

a file descriptor is a tool for the process to interact with the kernel, to the streams in particular


:::info
 Stracing strace
:::

1) Run “strace /bin/pwd” and save the result
2) Also run “strace strace bin/pwd” and save the result.
3) How do they related

Moreover, we can trace the process of tracing itself. In order to do that we can simply run this line
```strace strace /bin/pwd```
The result of this command saved in the file named ```'stracing strace'```
We will not show results to both tracings, but we can compare them
Firstly - the length of files
strace 45 lines
stracing strace 787 lines
It explanes by that we need much more system calls to every step of the process or recursive capturing
For example  this line 
```write(1, "/tmp\n", 5/tmp)               = 5```
replases with bigger one
```write(2, "write(1, \"/tmp\\n\", 5", 20write(1, "/tmp\n", 5) = 20 ...```
write(2) in this line is an SDRERR messedge, and tells that our terminal gives out an error


:::info
 ELF format
:::

1) Execute “readelf -Wh ping”

Let's find some information about the ping command.
For this, we will use following command
```readelf -Wh ping```
Where ```w``` means wide output, ```h``` for header

<center>

![](https://i.imgur.com/BR8LY5j.png)
Figure 3: Out put of ```readelf -Wh ping```
</center>

2) Match the results for the ELF header with the information on ELF’s Wikipedia page. Is this a definitive source for the ELF format?

Our output matches 
```7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00```
with we header on the wiki
```7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00```

This is the header of the executable ELF file
```7f 45 4c 46```
The next line is the class of the system, which means the 64-bit system
```02```
This line stands for the type of data, LSB (little-endian) in our case
```01```
The next line is also 01 but it means a different thing, it actually stands for the version of ELS
```01```
There is also information about currently using os and version of Application Binary Interface, type of machine that program on, and others like memory information.

3) inspect the section and program headers, using the command “readelf -WlS ```binary```”

a) Which of the two header types (section or program) is used in
which context (loader/runtime or linker/relocation)?

Let's look at the output of the ```readelf -WLS ping``` command
In the table section headers, we might see some processes with a different types.

<center>

![](https://i.imgur.com/uT5NWPL.png)
![](https://i.imgur.com/2wsnHOC.png)

Figure 4: headers of the ```ping``` command
</center>

Output can be divided into two blocks: program header and section header
Program header, it have been created in order to specify the process of process creation, and it, in our case, consists from:

PHDR - header itself
INTERP - information about interpreter
LOAD - segment of the memory that can be loaded
DYNAMIC - for the dynamics linking
NOTE - additional information about images
GNU_PROPERTY - needs for the programmes as AArch64 BTI or intel CET
GNU_EH_FRAME - part of the code segment
GNU_STACK - header that tells the system how it should be controlled
GNU_RELRO - part of the data segment

section header - allows us to locate every part of the file, and it looks like a table

NULL - unused part
PROGBITS - Program data
NOTE - same as in program header
GNU_HASH - hash table
DYNSYM - Dynamic linker symbol table
STRTAB - String table
VERSYM - version information
VERNEED - version Requirements
RELA - notes about movements
INIT_ARRAY - Array of constructors
DYNAMIC - same as in program header
NOBITS - absent of data

Everything in these headers can be divided into 4 subgroups:

linkers - program that combines several files into a single executable, like a library, parts of the program executed from the memory
relocation - allows to connect several programs, by loading (relocating) them into a new location
runtime - an environment that requires by the program in order to be executed
loader - reads contents of a file containing into memory, and prepare executable for running, so loads the program

b) Explain the use of each of the following sections: .text, .data, .rodata and .bss

Name or the header that indicates the type of the attribute also different, they are:
.text - contains executable instructions for the program
.data - part of data that inserted into the program memory
.rodata - part of data that is available only for read processes
.bss - this section holds data that affects the image of the program, it equals to the zeros at the start

System use this header to identify the type and attributes of sections

c) What does the program header with Type INTERP contain?

.interp - this one is containing information about the interpreter, relocation to the interpreter 
It consists of ```SHF_ALLOC```,  that identifies memory location during process of execution

4) Read your favourite ELF binary with hexdump utility

a) show virtual address and physical address.

physical address - the program in this case actually knows the layout of the RAM memory
virtual address - in case of the virtual memory, program executes in virtual environment, and operates with page table, the program uses it to interact with the memory

<center>

![](https://i.imgur.com/6Kbv2tX.png)

FIgure 5: Proggram header
</center>

On the hexdump of the ```ping``` binary we can find a lot of information, like the virtual and physical address
in our case lines, correlate with:
Virtual address
```40 00 00 00 00 00 00 00```
Physical address
```40 00 00 00 00 00 00 00```

In the binary file, there is more than a program header, there are several section headers.

<center>

![](https://i.imgur.com/8HSTDHj.png)

FIgure 6: Seccion header
</center>

 For the section header some lines are different:
Virtual address
```40 00 00 00 00 00 00 00```

In the binary file, there is more than a program header, there are several section headers.

b) show/explain the entry point of the ELF binary.

Entry is some kind of flag that tells the user that the new section starts, also Entry gives information like section name, size. In our case (64 bit system) entry is ```p_flags```, and equals 40

<center>

![](https://i.imgur.com/p1I1f64.png)

Figure 7: Entery point of the elf sile
</center>


5)  Find the loaded libraries of your shell by executing “lsof -p $$”

In order to list libraries that loaded of your shell, the user might use ```lsof``` with the key of not list PID's with "strange" numbers, like ```123,^456```

<center>

![](https://i.imgur.com/ZALcpYi.png)

Figure 8: List of loaded libraries

</center>

The figure 7 listed some currently loaded libraries, their unique ID, a user that have created them, used file descriptor, their type, size of the library, and name of the directory


5.1) Also, look at the memory image of your shell by executing “cat /proc/$$/maps”.

Memory image is another useful tool in system administering. The result of the ```cat /proc/$$/maps``` might be seen in figure 8

<center>

![](https://i.imgur.com/M4X3cK3.png)

Figure 9: List of processes in virtual memory

</center>

Every row in the output has such field as:
address, permissions,offset,device,inode,directory name
The out itself describes location of the memory occupied by the process or thread

5.2) Why is for instance the libc library mapped into memory multiple times?

Amount of calls to the library depends on the number of processes that uses this library. This exists because the pointer can only refer to the address space of the process.


---
## Task 4: The gcc compiler

:::info
 Hello SNE!
:::

Let's look through the process of the program compilation. To this end, we will create a new program: ```hello.c```
content of the program:

```
#include <stdio.h>
int main (int argc, char *argv[])
{
 printf("Hello st10!\n");
 return 2121;
}
```

It's a simple ```hello world``` program, written in C language.

1) Run “gcc -E hello.c -o hello.i”, producing the file “hello.i”.
What are the numbers at the end of the lines in “hello.i” supposed to mean?

```gcc -E hello.c -o hello.i```

This line executes GNU compiler collection with the -E (stop after preprocessing stage) and -o (place output in the file) Keyes

At this stage text replacements, cutting comments and including files are proceeding
In the new ```hello.i``` file library initialization process happen.
And numbers at the end of some lines mean
```# 454 "/usr/include/x86_64-linux-gnu/sys/cdefs.h" 2 3 4```

1 - the start of the file
2 - the return the file
3 - this text from the header
4 - implicit outer block

2) Run “gcc -S hello.i”, producing the file “hello.s”
Inspect “hello.s”. Where has “printf” gone?
Remove the newline at the end of the string in “hello.c” and look again at the
resulting “hello.s”. Explain.

At this stage, we get assembler code as an output of the
```gcc -S hello.i```
In the ```hello.s``` assembly file, for printing responsible this line
```
call	puts@PLT
```

where ```puts@PLT``` is a link to the library

if we dilete ```\n``` in the ```hello.c file```
In this case we are able to see ```print``` in the ```hello.s``` file (figure 9)

<center>

![](https://i.imgur.com/zVee0d7.png)
Figure 10: ```Hello``` program with and without  ```\n```

</center>



3) Run “gcc -c hello.s”, producing the file “hello.o”.
Use “objdump -rd hello.o” to inspect this relocatable object file.

Object file - in this file executable code is compiled but not linked, the output of the assembler.
Object files don't content several executable things, such as:
headers (linker adds header)
entry points (linker adds entry)
links to the other modules (linker adds does this)
That is why the CPU cant execute object file


4) Run “gcc -o hello hello.o”, producing the file “hello”.
Use “ltrace ./hello” to see what library calls your program makes.
Run “ltrace -S” to also see the system calls.
What is the actual exit code of your program? Can you explain this?

Let's  inspect our ```hello``` file with the ltrace

<center>

![](https://i.imgur.com/ZMhbVz3.png)
Figure 11: ltace of the ```hello```

</center>

Exit code of the ltrace depends on the return value in the program:
return = 2021
exit = 229

return = 257
exit = 1


:::info
 Stripped and non-stripped binaries (bonus)
:::

1) stripped vs non-stripped baniries (pros and cons)

stripped
+ smaller size
+ wihgher performanse speed
- dont have debbug info in it

non-stripped
+ have debbug info in it
- bigger size
lower effecensy of the

2) Compile code with stripped and non-stripped baniries

Lits watch the difference between stripped and non-stripped binaries of the code listed below.
You can see headers of the both files on the figure 12

```
#include <stdio.h>
int main(int argc, char *argv[])
{
printf("Stripped and non-stripped test\n");
return 0;
}
```

<center>

![](https://i.imgur.com/8KpCGqQ.png)
![](https://i.imgur.com/U2yKzgX.png)![](https://i.imgur.com/ONgxewE.png)
Figure 12: Stripped and non thripped binary

</center>


3) find entry point in a stripped binary file

One of the section of the output ```readelf -Wh sns-s``` tells us where is the entry point address of the program is located: 0x1060. On the figure 13 you may see output of the entry

<center>

![](https://i.imgur.com/DV8u3Bg.png)
Figure 13: header of the ```sns-s``` file

</center>

---
## Task 6: The file descriptor (Bonus)

:::info
1) Why do we need file descriptors?
:::

File descriptor (FD) - unique identifer for a open file\stream and information about file\stream. [14]

:::info
2) What is a file descriptor table in a process and what items it holds?
:::

File descriptor table describes the behaviour of the file: reading, writing, appending. Parts like opened process, its behaviour, and a lot more.
[14]

:::info
3) list file descriptors of  current bash session.
:::

We can use this command ```lsof -p $$ 2>/dev/null | awk '$NF ~ /\/pts\//'``` for listing Current bash descriptor session, where key actually means
lsof -p - open listing files with the clear readable PID.
awk - utility for the output changing
NR - the amount of the strings
$ - link to the column

<center>

![](https://i.imgur.com/W7TCEu9.png)
Figure 13: Current bash descriptor session 
</center>

The result can be interpreted as

0u in /dev/pts/1 - stdin for standart input
1u in /dev/pts/1 - stdout for the standart out
2u in /dev/pts/1 - stderr for the standart error
255u in /dev/pts/1 - process created by the bash itself


:::info
4) Where are the locations of file descriptor
:::

file descriptors stored in the ```/proc/PID/fd/```

<center>

![](https://i.imgur.com/HgRTgYQ.png)
Figure 14: directory  with the file discriptors
</center>

:::info
5) Write a code that creates a file and print the default file descriptors index and pointers in the file descriptor table of the process.
:::

:::info
Redirecting file descriptors.
:::
Explain and provide examples of redirecting file descriptors  (run a command)

In the shell, we can do more than just the execution of the program. Redirecting allow us to the user execute several programs in a chain.
Ther are sevreal types
Redirect to a file >
Redirect from a file <
Pipe |


for example:
```echo < filename``` means that we will take information from the file and print it

Example of simple rediracton:

<center>

![](https://i.imgur.com/nxNuy87.png)
![](https://i.imgur.com/KqTAjWd.png)
Figure 15: Script that redirect FD id's to the text file
</center>

---
## References:

1: [Unified Extensible Firmware Interface (UEFI) 
Specification](https://uefi.org/sites/default/files/resources/UEFI_Spec_2_8_final.pdf)
2: [Unified Extensible Firmware Interface](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface#Boot_stages)
3: [GNU GRUB](https://en.wikipedia.org/wiki/GNU_GRUB#Operation)
4: [Grub2/Setup](https://help.ubuntu.com/community/Grub2/Setup)
5: [Find out what filesystems supported by Linux kernel](https://www.cyberciti.biz/tips/tell-what-filesystems-linux-kernel-can-handle.html)
6: [EFI System Partition](https://wiki.osdev.org/EFI_System_Partition)
7: [Bare Bones](https://wiki.osdev.org/Bare_Bones#Booting_the_Kernel)
8: [EFISTUB](https://wiki.archlinux.org/title/EFISTUB)
9: [GRUB](https://wiki.archlinux.org/title/GRUB#Master_Boot_Record_(MBR)_specific_instructions)
10: [UEFI](https://wiki.osdev.org/UEFI)
11: [Booting](https://wiki.ubuntu.com/Booting)
12: [systemd man](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
13: [Rethinking PID 1](http://0pointer.de/blog/projects/systemd.html)
14: [File descriptor](https://en.wikipedia.org/wiki/File_descriptor)