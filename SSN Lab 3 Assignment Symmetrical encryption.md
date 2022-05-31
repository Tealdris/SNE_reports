###### tags: `SSN`
:::success
# SSN Lab 3 Assignment: Symmetrical encryption
**Name: Radomskiy Ilia**
:::

----
## Task 1: DES

:::info
1. Next, use the DES simulator (launch .jar file). Step through the process of encrypting your name with the key 0x0101010101010101 and write the internal state of the device at the 8th round.
:::

The first thing I have done is download a file and run it with `java DEScalc`

<center>
    
![](https://i.imgur.com/zMolM10.png)
</center>

Input for this attempt is:
Text
```
plaintext ilyailya
hex 696c6961696c6961
binary 110100101101100011010010110000101101001011011000110100101100001
```
key
```
hex 0101010101010101
binary 0000000100000001000000010000000100000001000000010000000100000001
```
Output:
```
hex 33c3955aade490cf
binary 11001111000011100101010101101010101101111001001001000011001111
```

On the 8th round we getting results listed below:
```
Right part of text 3fdfa190 
Left part of text 7fbd72de
key 0000000000000000
```

:::info
2. Inspect the key schedule phase for the given key and explain how the subkeys  are generated for each of the 16 steps.
:::

In the picture, you can see the process of key generation for DES, for step 1, but because of the key itself, process, and a key doesn't change.
All keys also were listed in the picture above

<center>
    
![](https://i.imgur.com/mqMsaeT.png)

</center>

:::info
3. Comment on the behaviour  of DES when using the given key.
:::

Key consisting of only zeros. It is happening because of the method of key generation, in the picture below you can see a representation of it. 8th bit is not used in the generation process, the yellow part is used for the `C` key, and orange for the `D` part.

<center>
    
![](https://i.imgur.com/6ZtcaAt.png)

</center>

---
## Task 2: AES

:::info
4. Identify the Shannon diffusion element(s).
:::

if you change 1 element of cypher text or plaintext, then your plaintext or cypher, respectively, have to change more than half of the symbols

:::info
5. Also identify the Shannon confusion element(s).
:::

1 element of plaintext should affect more than one element of cypher text, as well as 1 element of the key, should affect more than one element of ciphertext

[Wikipedia - Confusion and diffusion](https://en.wikipedia.org/wiki/Confusion_and_diffusion)

---
## Task 3: RC4

:::spoiler
Follow these instructions, identify the URL and your personal archive accordingly, download it and inspect its contents. There are two files encrypted with the RC4 cipher.
One of the files was encrypted using a 40 bit key that when represented in ASCII starts with the character a and contains only lowercase letters while the other uses a 48 bit key that can be written only with digits. Identify the encrypted files and using the brute force tool from this gist find the keys and decrypt the file. (Most likely you will have to install the pycrypto and numpy libraries first.)
:::

:::info
6.a. How did you identify the encrypted files?
:::

Only two files in the directory have entropy bugger than 6, it is `06279841.in` and `41579283.in`, that is why we can assume that it is an encoded file

<center>

![](https://i.imgur.com/Gm5Kc8i.png)

</center>

After that, we can try to brute-force them

:::info
6.b. What is the effective key strength for each of the keys?
:::

For decryption, we will be using [This python code](https://gist.github.com/cosu/4017169)
On the two pictures below you can see brute-force results, first for the `41579283.in` file and second for the `06279841.in` file.  As result, you can see the key, entropy, name of the file, and amount of time that was spent on Bruteforce

<center>
    
![](https://i.imgur.com/Li6mQoS.png)

![](https://i.imgur.com/xL65SM2.png)

</center>

The same information is also listening at the table.

| FILE | KEY | effective strength(binary) |
| -------- | -------- | -------- |
| 41579283     | aolpf     | 2^40     |
| 06279841     | 875136     | 2^48     |


Decrypted text you can see in the picture below

<center>
    
![](https://i.imgur.com/2M341AH.png)

</center>

[Sourse of plaintext information](https://www.gutenberg.org/files/2009/2009-h/2009-h.htm)



:::info
6.c. Instrument the code to find out how many decryption attempts you can perform in one second. Where is the most time spent ?
:::

Bruteforce algorithm is working with a particular speed (keys per second), so in order to see, what kind of speed, I have expanded python code with mine, which is listed below

<center>
    
![](https://i.imgur.com/CcIs84u.png)

</center>

<center>
    
![](https://i.imgur.com/BZsFaRc.png)

</center>

From the picture, we can say that in one second there are approximately 2500 attempts

:::info
6.d. Modify the code to support parallel execution and calculate the speedup.
:::

In order to make code execute in parallel mode, I changed several parameters, amount of cores, involved in the execution process, and lines listed in the picture below

<center>
    
![](https://i.imgur.com/oEIoBxa.png)

</center>

Now, you can see the difference in time of execution, it is reduced, and it is represented on the second of the paired pictures below

<center>

![](https://i.imgur.com/Li6mQoS.png)
![](https://i.imgur.com/x5LrOXU.png)

</center>>

<center>
    
![](https://i.imgur.com/xL65SM2.png)
![](https://i.imgur.com/elBSNrm.png)

</center>
    
And on this picture, you can see how many attempts can be performed in one second. It is about `1500*4`
    
<center>
    
![](https://i.imgur.com/qieoeGx.png)

</center>
    
:::info
6.e. If the same message would be encrypted with a key of length 48 bits but which uses all the printable characters, how much time would it take to explore the full key space ?
:::


According to the picture below (representation of ASCII), there are 95 printable characters, so in the table listed below, you can see the number of attempts, for this, and scenarios above

![](https://i.imgur.com/LlLX1fP.png)

| Amount of characters | Password lengt(characters) | Password lengt(bits) | Time  | Possibilities |
| -------- | -------- | -------- | -------- | -------- |
| 26     | 5 | 40 |  79 min    | 11881376                             |
| 10     | 6 | 48 | 6 min      | 1000000                              |
| 95     | 6 | 48 | 9.5 years  | 735091890625 |


---
## Task 4: AES

:::info
7. Modify the code to support AES brute-force in CBC mode. How many keys can you test per second? 
Julian Assange has released an insurance file encrypted with AES256. Assuming
that no disruptive technological breakthrough will take place in the future and the performance of CPUs will double every 18 months, when will it be possible to brute-force the file in reasonable time, i.e. less than 1year, using a single computer?
:::

If we want to use python brute-force for AES decryption, we have to change several lines in the application

<center>
    
![](https://i.imgur.com/CXlP1nh.png)
    Libraries
![](https://i.imgur.com/tMoK6Hp.png)
Code specificatoin line
    
</center>

The process of decoding is shown in the pictures below, on the first picture there are lines with the key attempt and entropy
In the second picture amount of attempts have done to a particular second. According to this number, we can say that we are trying about `25000` keys every second. 

<center>

![](https://i.imgur.com/c8C3Z3p.png)

</center>

<center>
    
![](https://i.imgur.com/zt5ukgI.png)

</center>

As we can see in the picture with the keys. It will take a lot of time to brute-force this key, the exact number is `26^32 = 1901722457268488241418827816020396748021170176` attempts with using ONLY lowercase letters
According to [this](http://xorloser.com/blog/?p=93) reference, it is not possible to brute force AES in a year. For my current speed rate it will take `1.472732107719223e+65` years for `AES-256`, but for the fastest quantum pc on the earth (Fugaku) it will take `8.307111002445255e+51` years.
This was counted with 
`2^256/(operations_per_sec*sec_in_year)`

But we can make an assumption on it.
I wrote i simple python program for it, it is shown in the picture. I will take my number of keys `2500` per seconds for starting point

![](https://i.imgur.com/NZCW6Rh.png)

As a result we can see, that it would take 212 years, to brutefoerse AES256, if the power of CPU's will increase in double every 18 month

![](https://i.imgur.com/ZXsZEYn.png)




---
## References:

[](paste -sd "" "06279841(copy).txt" > "06279841(copyn).txt")