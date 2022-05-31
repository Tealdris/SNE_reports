###### tags: `OT`
:::success
# OT Lab 1 Assignment: Assembly Patterns
**Name: Radomskiy Ilia**
:::

---

## Part 1 - Understanding Assembly

### Task 1: Preparation

:::info
1. Choose any debugger/disassembler what supports the given task architecture (32bit & 64bit). **Ghidra**
:::

```
sudo apt update && sudo apt upgrade
sudo apt install default-jdk
./ghidraRun
```


:::info
2. You have to check and analyze what does a binary do before running it. You have no idea what file we are working with, even if it is initially considered as legitimate application.
:::

Got it

:::info
3. Prepare a Linux VM for this lab, as you might need to disable ASLR
:::

I will be using KVM virtualization in `VMM` with the `ubuntu20.04` installed in it

### Task 2: Theory

:::info
1. What is the difference between **mov** and **lea** commands?
:::

**mov** - Load Value or Moves data (actual value)between registers and memory


**lea** - Load effective address or loads a pointer to the item

<center>

![](https://i.imgur.com/2WUfZfx.png)
Difference between MOW and LEA
</center>


    
:::info
2. What is ASLR , and why do we need it?
:::

**ASLR** - address space layout randomization, is used in order to make memory layout unpredictable, in order to prevent access to this memory part. complications of exploitation of several types of vulnerabilities like buffer Overflow and others

:::info
3. How can the debugger insert a breakpoint in the debugged binary/application?
:::

[C++](https://www.youtube.com/watch?v=0DDrseUomfU) specified according to the source

There 2 types of breakpoints
**Hardware**
The special register will result in breaks
Limited:
* 4 registers DR0-DR4 writing breakpoints
* DR4/DR6 debus status register
* DR5/DR7 debug control register
Can break on read\write\execute

**Software**
Modify the running code in memory to introduce breaking points
Unlimited
Can break on executing

If you want to set up software breakpoint inline
```bash=
48 89 e5        mov %rsp, %rsp
```
The debugger will take 1 bite `48` and replace it with the new value ox`cc` this is instruction and it triggers software interruption
```bash=
48 # will be in another place
cc 89 e5        mov %rsp, %rsp
```
after that signal called `SIGTRAP` is telling to a process to make a stop
```bash=
SIGTRAP    5    Core    Trace/breakpoint trap
```

So the program that you want to be debugged in is running "inside" debugger. At this time debugger is in the `wait` stage. After we reach a place of a breakpoint with the new `cc` bit value. It redirects us to the `do_trap` function with `SIGTRAP` in it.
After `SIGTRAP` is got, the debugger awakes.

### Task 3: Disassembly

:::info
1. Disable ASLR in your Linux VM using the following command:
:::

```
sudo sysctl -w kernel.randomize_va_space=0
```

From this point in the VM there is no randomisation

<center>

![](https://i.imgur.com/M4MHn8B.png)
Config file
</center>

:::info
2. Load the binaries ( sample 32 and sample 64 from part1.zip) into a disassembler/debugger
:::


For the 32-bit and 64-bit applications, analyzed in `ghidra` we can see such output

<center>

![](https://i.imgur.com/Vp02zza.png)
Results summary of 32 binary
</center>

<center>

![](https://i.imgur.com/y18ZTkv.png)
Results summary of 64 binary
</center>

As we can see from the pictures ghidra has determined its type, compiler, size of the address, and format of the executable. this information will be useful for us later on.

:::info
3. Does the function prologue and epilogue differ in 32bit and 64bit? What about calling conventions?
:::

`Prologue` - part of the machine code in the function, purposed to reserve registers and stack for the normal execution. Usually located at the beginning of the machine code

`Epilogue` - part of the machine code in the function, purposed to release registers and and stack after execution. Usually located in the end of the machine code

[`Calling conventions`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/x64-architecture) - set of rules that are used when the function is called.
In `GCC` main thing is
`caller` - something is calling, main is usually called
`called` - something is being called, may be called as a part of the main

We can find function prologues and epilogues of the binary in `ghidra`. Picture of them is located below for the 32-bit application.


<center>
    
![](https://i.imgur.com/2ioO6xC.png)
Prologue for 32-bit file
</center>

<center>

![](https://i.imgur.com/NPYYG8X.png)
Epilogue for 32-bit file   
</center>

As you can see in the pictures above, main reason for the existence of both, prologue and epilogue is preparation and cleaning. In the Prologue, we can see `PUSH` commands, exactly 4 of them. By calling them we reserve registers `EBP`, `EDI`, `ESI`, `EBX` with our programs, which make our program holistic.
In the epilogue, we are releasing those registers with the `POP` command. This part is required not to leave the register locked, and for the availability of the others programs.

In the example below you can wear both the Prologue and epilogue of the small function, in this case, only one register is being used by the function

<center>

![](https://i.imgur.com/Wshmbqj.png)
Prologue and epilogue of the small function    
</center>


In the picture below you can see calling convention `cdecl`. This caller have full name of `c-declaration` and, of course it is called from `C` language of x86 compilers.

<center>

![](https://i.imgur.com/E0yYKNU.png)
Calling convention of 32-bit exe

</center>

In the section below you can see the prologue of the 64-bit application. There is not much difference compared to the 32-bit application. `PUSH` command is also used in order to use registers.

<center>
    
![](https://i.imgur.com/MyDNbyn.png)
Prologue for 64 bit file
    
</center>

The epilogue of the 64-bit application is also not that different from the 32-bit application.
But both prologue and epilogue are longer comparatively to the 32-bit exe

<center>
    
![](https://i.imgur.com/zq8fhzL.png)
Epilogue for 64 bit file
    
</center>

In the example below you can see both the Prologue and epilogue of the small function, in this case, only one register is being used by the function

<center>
    
![](https://i.imgur.com/rJFsKbY.png)
Prologue and epilogue of the small function of 64-bit application
    
</center>

The `calling convention` of the 64-bit application is different. Different call `sdtcall` is used. There is no calling convention `cdecl` in the 64-bit application.

<center>

![](https://i.imgur.com/qlLafXv.png)
Calling convention of 64-bit exe
    
</center>


:::info
4. Does function calls differ in 32bit and 64bit? What about argument passing?
:::



A [Function calls](https://www.cs.princeton.edu/courses/archive/spr11/cos217/lectures/15AssemblyFunctions.pdf) is when `mian` function (`caller`) addresses another function (`calle`) by its memory address of the first instruction. After `call` is used, execution is going back or returning in the place of the call
If the `called` is used from multiple sources, registers may be used to solve this returning situation.
If there is a nesting of functions, then stack memory can be used to store values of the returning point

To pass some parameter to function (`caller` -> `callee`) we might use registers. But there is a problem with passing parameters longer than 4 bytes (32 systems).
The main difference between 64 and 32-bit systems is the number of registers and their length. That is why there are more registers used during the execution of the program and operation happens quicker in 64-bit operation systems
In order to solve this issue `stack` can be used. Pushing of parameters happens before executing call instructions, and parameters are pushed in reverse order (1 parameter is on the top of the stack).
After parameters is being used we can `pop` them from the stack 

:::info
5. What does the command `ldd` do? “ ldd BINARY-NAME ”.
:::

`ldd` shows shared objects\libraries required by each program or on the command line.

<center>

![](https://i.imgur.com/rcIuBAO.png)
ldd command
</center>


I received `not a dynamic executable` because I tried to execute this x86 file in x64 bit [linux](https://unix.stackexchange.com/questions/75054/ldd-tells-me-my-app-is-not-a-dynamic-executable)

---

## Part 2 - Reverse engineering

### Task 1: Theory
:::info
1. What kind of file did you receive (which arch? 32bit or 64bit?)?
:::

Binaries 1 -6 are x86-64 architecture

<center>

![](https://i.imgur.com/kjcWgxF.png)

</center>

:::info
2. What do stripped binaries mean?
:::

`Non-stripped` binaries have debugging information built into it
`Stripped` binaries don't have to debug information built into it
As for binary 1, we can find the `main` function in this window of the `ghidra` application. it is a `non-stripped` binary, that is why there is debugging information

<center>

![](https://i.imgur.com/MMppDDu.png)
    
</center>

In the previous section we can see that, binaries 1-4 are `non-stripped`, 5 and 6 `stripped`

:::info
3. What are GOT and PLT ? Describe on the practical example.
:::

`PLT` Procedure Linkage Table put a link to a memory location without knowing is it there or not, dynamic linker will resolve it

`GOT` Global Offsets Table this table is used when there is no address for  your data or function in  code, is used for the resolving

In the picture below you can see how the resolution happens. When the linking happens for the second time, resolved address will be stored in `GOT`

<center>

![](https://i.imgur.com/LTt1jU1.png)
Resolution of link
</center>

On the pictures below we can see whole precess of the linking. This is from the binary of the simple `helloworld.c` file

First of all we jumps from the main function to the `puts`, this is functopn of printing, and it is a part of the standart c library. Lince our binary file if dynamicaly linked, this will be resolved by the `PLT`

<center>

![](https://i.imgur.com/hBGrBDh.png)
Main function
</center>

Now, after jump, we are in the `PLT`, but, soom after we jumps to the `GTO` (follow the red lines)
<center>

![](https://i.imgur.com/QtxcjeZ.png)
Jump from the main to the PLT
</center>

In the GOT section we are, finaly, and the end of our adventure, and sucesefuly resolved `puts` function

<center>

![](https://i.imgur.com/Vz9Ieca.png)
Jump from the GOT to the GOT    
</center>


:::info
4. What are binary symbols in reverse engineering? How does it help?
:::

Those symbols can help you understand the program's ability to work. They are present in the table below

| Class | Explanation |
| -------- | -------- |
| a/A     | absolute symbol     |
| b/B     | bss (that is, uninitialized data space)symbol     |
| C     | common symbol     |
| d/D     | data(this is, initialized data space) symbol     |
| I     | indirect reference to other symbols     |
| N     | debugging symbol     |
| r/R     | read only data section     |
| t/T     | text symbol     |
| U     | undefined symbol     |
| v/V     | weak symbol     |
| w/W     | weak symbol(tagged)     |
| ?     | unknown symbol     |

In the picture below we can see those symbols as a part of the `bin1` 

<center>

![](https://i.imgur.com/27yQQTV.png)

</center>

---

### Task 2: Reverse & Cracking

:::info
Inside the ZIP file (**part2.zip**), you will have multiple binaries (`bin 1-6`), try to reverse them by recreating them using any programming language of your choice (C is more preferred). You are given two simple PE files as well:
:::

My st number `st 10`, so i will be working with the binaries only

:::info
* task1.exe (it was found that this binary is dynamically linked to the debug version of Microsoft Visual C++ redistributable 2015. You can put this package together with Visual Studio installation, or add dll file that is inside the zip archive)
* task2.exe
:::

:::info
Your task is to crack the program and find the correct password.
**Tasks distribution:**
if your **st** number is even: you have to work with bin 1-6
if your **st** number is odd: you have to work with bin 1-4 + task1.exe + task2.exe
:::

In the sections below I will be analyzing the raw binary of C (possible) code. Why C, because it was specified in the task

#### **binary1**

We can see a graphical representation of bin 1 in the picture below. The last part of the picture is an if statement and it is called `cannary`  it saves from the unappropriated data addressing and such exploit as bufferoverflow.

<center>

![](https://i.imgur.com/oKnnPr0.png)
Functional graph
</center>

As we can see in the picture below two variables are called and such parameter as `local time` is given to them. As name of it stands it return local time.

<center>

![](https://i.imgur.com/FhH8lFw.png)
Assembly commands
</center>

Right here, on the picture below, function `printf` is called the area is highlighted with a red marker. Inside, there are several arguments, particularly 6 of them. We can find all of the little bit above, in red rectangular. Those are parameters of the localtime function of the library `time.h `. I display the parameter and its description in the table below
    
| Parameter | Discriiption |
| -------- | -------- |
| tm_sec     | seconds     |
| tm_min     | minutes      |
| tm_hour     | hours      |
| tm_mday     | day of month     |
| tm_mon     | month     |
| tm_year     | year scince 1900     |
| tm_wday     | Day of week     |
| tm_yday     | Day of year     |
    
<center>

![](https://i.imgur.com/ykpCde0.png)

</center>

`Ghidra` suggests a possible code, that was used in order to make this kind of binary. In the picture below there is a red rectangular with the variables and the statement of the `cannery`. And, if the variables `local_10` and `in_FS_OFFSET` are not equal, the program will not be executed. The canaries added to the program by the compiler, so we don't have to write them in `C`

<center>

![](https://i.imgur.com/PJvCLTh.png)
    
</center>


Below there is code in `C`. We can prove that the binary from its code is the same as the original one, by analyzing it in `ghidra`.

```c=
void main()
{
    time_t t;
    struct tm *local;

    t = time(NULL);
    local = localtime(&t);
    printf("%04d-%02d-%02d %02d:%02d:%02d\n",(ulong)(uint)local->tm_year,
        (ulong)(uint)local->tm_mon,(ulong)(uint)local->tm_mday,(ulong)(uint)local->tm_wday
        ,(ulong)(uint)local->tm_min,(ulong)(uint)local->tm_yday);

    return 0;
}
```
This code will take local time and print it out.
**0122-03-01 05:31:90**
**0122** since 1900
**03** - April, if January is 0
**1** day of the month
**5** day of the week (Friday)
**31**-st minute, range 0 to 59
**90** days in the year, range 0 to 365

Ghidra provided the code, close to the original binary
<center>

![](https://i.imgur.com/aqgq4OO.png)
Suggested C code by ghidra
</center>

Graphical representation of binary is also the close to original
<center>

![](https://i.imgur.com/Z6TCMTQ.png)
Graphical representation of binary
</center>

And the output of them, both of binary files the same.
<center>

![](https://i.imgur.com/aCl68t7.png)
![](https://i.imgur.com/i6YH4Mi.png)
Output of the binaries
</center>


#### **binary2** && **binary3**

The output of the files is the same, so i might suppose that the binaries are doing the same thing by one algorithm, and we will investigate further on it.

<center>

![](https://i.imgur.com/qRtnlXW.png)
The output of the binary 2 and 3
</center>

<center>

![](https://i.imgur.com/iEzQTa9.png)
![](https://i.imgur.com/Vf3BuO8.png)
Graphical representation of the function's logic of the 2 and 3 binary
</center>

From this point, we might consider comparing them with some tools like `diff` or/and `cmp`. In the two pictures below we can see - there is no difference between binary 2 and 3, compared to binary 1 and 3.

<center>

![](https://i.imgur.com/0GbNY5l.png)
![](https://i.imgur.com/uHjKFwI.png)
Binary comparison
</center>

From this point, we will investigate further them as one binary.

As in the previous binary, at the end of this one, there is a protective `canary`.

<center>
    
![](https://i.imgur.com/LXNSAHU.png)
Cannary as a block scheme
</center>

First, the `main` part is used in order to set up the environment for the program, from it we `jump` to the fist (LAB_001006dd) `if` condition check, if the number is less than 19(0x13), we are going into the `while` loop (LAB_001006ca), in the loop.
Inside the loop we
```
copy from the RBP with the offset           MOV        EAX,dword ptr [RBP + local_6c]
add var to itself                           LEA        EDX,[RAX + RAX*0x1]
copy from the RBP with the offset           MOV        EAX,dword ptr [RBP + local_6c]
Convert Doubleword to Quadword              CDQE
Copy in new format from the register EDX    MOV        dword ptr [RBP + RAX*0x4 + -0x60],EDX

adding 1 to the variable                    ADD        dword ptr [RBP + local_6c],0x1
```

<center>

![](https://i.imgur.com/jUDICku.png)
First loop
</center>

After the first condition is no longer valid, we can go to the next loop, `JMP` to the `LAB_0010070f`, or to the `if` statement and `while` loop `LAB_001006ec`. In the loop we are, by increasing variable and printing information from the array

```
copy from the RBP with the offset            MOV    EAX,dword ptr [RBP + local_6c]
Convert Doubleword to Quadword               CDQE
to EDX from 2 reg with offset (new format)   MOV    EDX,dword ptr [RBP + RAX*0x4 + -0x60]
copy from the RBP with the offset            MOV    EAX,dword ptr [RBP + local_6c]
Copy from EAX to ESI                         MOV    ESI,EAX
Load this as format of the printed data      LEA    RDI,[s_a[%d]=%d_001007b4]
Set EAX to 0                                 MOV    EAX,0x0
call function printf                         CALL   <EXTERNAL>::printf 
adding 1 to the variable                     ADD    dword ptr [RBP + local_6c],0x1

```

<center>

![](https://i.imgur.com/xi2AwhV.png)
Second if statement    
</center>

This code is a result of the binary analysis
```c=

int main(void)

{
  uint x;
  uint array [22];

  for (x = 0; (int)x < 20; x = x + 1) {
    array[(int)x] = x * 2;
  }
  for (x = 0; (int)x < 20; x = x + 1) {
    printf("a[%d]=%d\n",(ulong)x,(ulong)array[(int)x]);
  }

  return 0;
}
```

It will do the following:
First `if` statement generates an array of the even numbers, iterable variable `x` increases every loop iteration by 1, this number is also an index number, position, in the array. This `x` also used to generation value in area array: [x] = 2x
The second `if` statement is used in order to print `array` out, the conditions of the loop are the same as the first one, start position of `x` is 0, it will be increasing by 1, until gets to 19, not more than 20.

In the picture below you may find graphical representation of the binary

<center>

![](https://i.imgur.com/uohfXSF.png)
Graphical representation of the function's logic of constructed binary
</center>

<center>
    
![](https://i.imgur.com/tOruUQo.png)
![](https://i.imgur.com/gyss6DF.png)
Code generated by ghidra from the analysis of 2 binaries, original one, and recreated
</center>

#### **binary4**

From the picture below we can see that 4-th binary is solving, and giving answer on a question, `is the number even or odd?`

<center>

![](https://i.imgur.com/kJbtnKE.png)
Output of the program
</center>

<center>

![](https://i.imgur.com/wTebpzc.png)
Graphical representation of the function's logic of binary
    
</center>

By taking a closer look at the binary and its graph we can say
In the beginning, we are printing `Enter_an_integer` from the register `RDI`, asking for the input with `scanf` and storing it in the variable or `RAX` register. After the initialization stage is finished, we start figuring out the parity of the number.
Then we need to calculate parity, in the program it is done simply, we do the `AND` operation with the number, after that we compare it with 0, if the result is equal to 0, then the number is `even`, otherwise is` odd`.

```
and operation         AND        EAX,0x1
comparison with 0     TEST       EAX,EAX
```


<center>

![](https://i.imgur.com/fnZHHi4.png)
Initialization stage with first print and input acquirement
</center>

In the section below there are only printing calls, for two conditions, `even` and `odd`

<center>

![](https://i.imgur.com/vGZv7Tv.png)
    
</center>

So, now we can code this logic in `C`

```c=
#include <stdio.h>
#include <stdlib.h>

int main(void)

{
  uint input;

  printf("Enter an integer: ");
  scanf("%d", &input);
  if ((input & 1) == 0) {
    printf("%d is even.",(ulong)input);
  }
  else {
    printf("%d is odd.",(ulong)input);
  }
  return 0;
}
```

In the picture below we can see output of the constructed binary

<center>

![](https://i.imgur.com/0Pzl87Y.png)
Output of the constructed binary
</center>

By analyzing this code in `ghidra` we can see, that the resulting binary is close to the original

<center>

![](https://i.imgur.com/WKDCnK8.png)
Graphical representation of the function's logic of constructed binary        
</center>

<center>
    
![](https://i.imgur.com/z6aUr4t.png)
![](https://i.imgur.com/p5LCC0S.png)
Code generated by ghidra from the analysis of 2 binaries, the original one, and recreated one
    
</center>

#### **binary5**

Again, analysis of the binary we will start with running it. Yes, it is dangerous, because it can be a malicious file, but I have a little bit of hope in our TA's.

<center>

![](https://i.imgur.com/7gEkdLy.png)
Output of the binary5
</center>

The input of the program is integer number >= 0
The output of the program integer number, equals the:  **output = input!**
Or, an **error** message

Since it is `stripped` binary we cant just use search by the functions. But I, since i couldn't find any other option, choose to search with a function call. Because, in the previous binaries, the same system call `stdcall` was also used
After I got the list of the sunction calls, i just go through them and have chosen to most `main` "look like" part 

<center>

![](https://i.imgur.com/E7uDv3o.png)
Search window
</center>

<center>

![](https://i.imgur.com/6j4TKrZ.png)
Graphical representation of the function's logic of binary 5

</center>

Let's take a closer look at them
First part similar to the same part from the previous binary
We occupy some of the registers, print a message about the need to enter data, and receive this input as a part of the `scanf` function call.
At the end of this part we check 1 condition, the input number must be more or equal to the 0 otherwise print an error message

<center>

![](https://i.imgur.com/JPMXUF1.png)
Initialization stage with first print and input acquirement
</center>

This 'print error message' block is shown in the picture below. If the error happens, when it is the end of the program
If the situation is smooth, we can proceed with the execution
In the `if` statement section (LAB_0010079a), we compare the iterable variable with the input. And if the variable (set to 0 in the beginning) is bigger than the input we stop execution.
In the section of the loop, where is factorial is calculated we use such formula to calculate factorial: 
`factorial_number = variable * factorial_number`

```
copy from the RBP with the offset    MOV        EAX,dword ptr [RBP + local_1c]
Convert Doubleword to Quadword       CDQE
copy from the RBP with the offset    MOV        RDX,qword ptr [RBP + local_18]
multiplication                       IMUL       RAX,RDX
copy from the RAX                    MOV        qword ptr [RBP + local_18],RAX
iterator 1 increment                 ADD        dword ptr [RBP + local_1c],0x1
```

Print happens similarly to the previous binaries.

<center>

![](https://i.imgur.com/l2LQhv3.png)
The main logic of the binary
</center>

Now we can code all of the features in `C`

```c=
#include <stdio.h>
#include <stdlib.h>

int main(void)

{
  uint input;
  int x;
  long factorial;

  factorial = 1;
  printf("Enter an integer: ");
  scanf("%d", &input);
  if ((int)input < 0) {
    printf("Error!");
  }
  else {
    for (x = 1; x <= (int)input; x = x + 1) {
      factorial = x * factorial;
    }
    printf("Result is %d = %llu",(ulong)input,factorial);
  }

  return 0;
}

```

The output of the Resulted binary looks like this

<center>
    
![](https://i.imgur.com/UnrZ9gW.png)
binary output
</center>

We also can compare two functional block schemes of binary, in the picture below you can see such graph of the built binary from the code, written by me.

<center>

![](https://i.imgur.com/dqeHo04.png)
Graphical representation of functions in binary
    
</center>

Also, the code of the binary, original and its copy really close to each other.

<center>

![](https://i.imgur.com/9PP1Pal.png)
![](https://i.imgur.com/DFDFVnm.png)
Comparation of the code suggested by ghidra
</center>

#### **binary6**

By running this programm we can see that this is simple program, takes 2 integers, summarize them, and print the answer in format of `input1 + input2 = sum`. Seemse simple enough. But analysis of raw binary takes unexpectedly long amount of time. 
<center>

![](https://i.imgur.com/zikeusx.png)
    
</center>

This is due to amout of bytes in this file. 171 times more that in binary5

<center>

![](https://i.imgur.com/tKG8CUm.png)
Window with the information about the binaries
</center>

It will be hard to look throug this file, especialy because it is `stripped`
But output of the programm gives us a hint: `Enter two integers:`
By this key word we can find main function.

<center>

![](https://i.imgur.com/L458Vok.png)
Result of the search
![](https://i.imgur.com/lNXTQmi.png)
Location of the founding
</center>

From this point we can start unraveling the logic behind how the binary works

<center>

![](https://i.imgur.com/1vIiyFg.png)
Main block of functions
    
</center>

Main function consist of 3 function calls and one mathematical expressions. By the lookinng on those functions we can say, those are function from the standart library of `C` language. That is why signatures of those libraryes can help us to identify them.
We can go the this [Github](https://github.com/push0ebp/sig-database) repository and download them. On the picture below we can see version of the ubuntu, whitch was used in order to compile this binary.

<center>

![](https://i.imgur.com/Wbal7iz.png)
    
</center>

Then we can inplement those signatures in the Ghodra with the script from this [Github](https://github.com/NWMonster/ApplySig) 
After that our functions are renamed

<center>

![](https://i.imgur.com/mssE3ep.png)
resolved function names
</center>

Now we can start analysis of this binary. Patrs of this binaty is realy close to the prevoius one
In this section we print out message, asking for input
<center>

![](https://i.imgur.com/jKQLsZt.png)
Printing message 'Enter two integers'
</center>

In this picture below we are ready to accept input with the function `scanf`

<center>

![](https://i.imgur.com/6tBBFjU.png)
Requesting input
</center>

This part is the biggest, but, still, ease, and realy close to the previous binaries, we adding 2 input values to each other, and print them in terminal

<center>

![](https://i.imgur.com/TYtaxcT.png)
Print result of the function
</center>

Last thing to do, make cannary check by calling canary function.
<center>

![](https://i.imgur.com/TbyhlHr.png)
Canary call
</center>

Now we can write similar code in `C`

```c=
int main(void)

{
  int input1;
  int input2;
  int summ;

  printf("Enter two integers: ");
  scanf("%d %d",&input1,&input2);
  summ = input2 + input1;
  printf((char *)"%d + %d = %d",input1,input2,summ);
  return 0;
}
```

But we will not get the same results to the binary6 or close to. Because bin6 is staticaly linked. GCC created dynamicaly linked binaries by default.

<center>

![](https://i.imgur.com/QOWcaJY.png)

</center>



To make binary staticaly linked one we have to specify it with

```
gcc -static main.c
```

And to strip biunary

```
strip a.out
```

After that we can analyse the binary. Dont know why, binaryes are realy close to each other, the only difference is amount of `undefined param_x`. We can see this on the example of the psedo code generated by ghidra. This difference is not only present in the pseudo code

<center>

![](https://i.imgur.com/FDX615Q.png)
    
</center>

But, when i tried to analyse this code in the IDA and result is normal.

![](https://i.imgur.com/uC0BljP.png)

This corresponds to the other students.

<center>
    
![](https://i.imgur.com/KRhfsRz.png)
![](https://i.imgur.com/TNrhkQw.png)
    
binary6 analysed by my goupmate Naghme
</center>


### Bonus

:::info
Describe the difference between PE and ELF executable files. Dhow it by a practical example
:::

I will show the difference between ELF and PE on the binary 6.

`ELF` - Executable and Linkable Format
`PE` - portable executable

We can create PE file with folowing command
```bash=
x86_64-w64-mingw32-gcc -o main64.exe main.c
``` 

<center>

![](https://i.imgur.com/1oY6gfH.png)
ELF and PE files
    
</center>

<center>

![](https://i.imgur.com/lZrFvbN.png)

</center>

And right from the beginig we can see difference

<center>

![](https://i.imgur.com/DKGptlF.png)
    
</center>

**ELF** files consist of the:
Header
Sections
Segmnets

Header of the ELS looks like this

```c=
#define EI_NIDENT 16

struct elf32_hdr {
  unsigned char e_ident[EI_NIDENT];
  Elf32_Half e_type;
  Elf32_Half e_machine; // processor info
  Elf32_Word e_version;
  Elf32_Addr e_entry;  /* Entry point */
  Elf32_Off e_phoff; //offset
  Elf32_Off e_shoff;
  Elf32_Word e_flags;
  Elf32_Half e_ehsize;
  Elf32_Half e_phentsize;
  Elf32_Half e_phnum;
  Elf32_Half e_shentsize;
  Elf32_Half e_shnum;
  Elf32_Half e_shstrndx;
};
// and inside e_ident
struct {
  unsigned char ei_magic[4]; //{ 0x7f, 'E', 'L', 'F'} 
  unsigned char ei_class;
  unsigned char ei_data; //LSB or MSB
  unsigned char ei_version;
  unsigned char ei_pad[9];
}
```



**PE** is close to the ELF:
00h - EXE header
20h - OEM header
3сh - offset PE
movement table `stub`;
`stub`;
PE header;
object table;
file objects; 

`stub` is and programm, it is executen in realtime.

Header of the PE looks like this

```c=
struct pe_hdr {
  unsigned long  pe_sign;
  unsigned short pe_cputype;
  unsigned short pe_objnum;
  unsigned long  pe_time;
  unsigned long  pe_cofftbl_off;
  unsigned long  pe_cofftbl_size;
  unsigned short pe_nthdr_size;
  unsigned short pe_flags;
  unsigned short pe_magic;
  unsigned short pe_link_ver;
  unsigned long  pe_code_size;
  unsigned long  pe_idata_size;
  unsigned long  pe_udata_size;
  unsigned long  pe_entry;
  unsigned long  pe_code_base;
  unsigned long  pe_data_base;
  unsigned long  pe_image_base;
  unsigned long  pe_obj_align;
  unsigned long  pe_file_align; 
  etc;
};
```

As we can see from the description abobe field are different, as well as the objects in it off course.

---
## References:


:::spoiler

Ghidra
https://daniao.ws/notes/quick-tips/build-ghidra

JDK 11

https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-20-04-ru

sudo update-alternatives --config java

Gradle (old)
https://andreyex.ru/ubuntu/kak-ustanovit-gradle-na-ubuntu-18-04/

https://tel.archives-ouvertes.fr/tel-01836311/document

to make exec 
gcc -no-pie OTTestC/bin6/bin6/main.c

:::