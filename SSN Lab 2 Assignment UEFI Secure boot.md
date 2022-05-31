###### tags: `SSN`
:::success
# SSN Lab 2 Assignment: UEFI Secure boot
**Name: Radomskiy Ilia**
:::


## Task 1: Firmware Databases
:::info
1. Extract the Microsoft certificate that belongs to the key referred to in Step 1 from the UEFI firmware, and show its text representation.
:::

Microsoft certificate can be extracted with several methods:

`efi-readvar -v KEK -s 0`
this line prints certificate with number 0 in the terminal, we also can save the output to the file, and print it in the terminal in hexadecimal format
You can see all of above in the picture below

<center>

![](https://i.imgur.com/vTxKzfK.png)

</center>

another method is to use `mokutil`, with the key `--kek` in our case
as output we getting all (2) available kek keys on the machine
You can see output on the picture below

<center>

![](https://i.imgur.com/7bxK7gH.png)
![](https://i.imgur.com/XJT9G1l.png)

</center>
so we got 
subject - Microsoft Corporation KEK CA 2011
issuer - Microsoft Corporation Third Party Marketplace Root

other possible  ways: `openssl x509`, etc

:::info
2. Is this certificate the root certificate are  the chain of trust? What is the role of the Platform Key (PK) and other variables?
:::

No, Microsoft Corporation certificate is a KEK (Key Encryption Key) certificate
You can see hierarchy of certificates in the picture below

<center>

![](https://i.imgur.com/RfuihQ3.png)

</center>

[About security UEFI](https://habr.com/ru/post/267953/)

We can find other keys that are  present on the pc

PK - used to create a trust relationship between the platform and its firmware, it is root key and it is self-signed.
subject - Hewlett-Packard UEFI Secure Boot Platform Key
issuer - Hewlett-Packard Printing Device Infrastructure CA

<center>

![](https://i.imgur.com/AEJx98h.png)
![](https://i.imgur.com/UiMZVYd.png)
PK key
</center>

There are other types of keys, DB and dbx, on the pictures below you can see partial output of them

DB - white list of keys

<center>

![](https://i.imgur.com/pICASSt.png)
DB keys
</center>


DBX - black list of keys

<center>

![](https://i.imgur.com/j6kBl6t.png)
DBX keys
</center>

---
## Task 2: SHIM

:::info
3. Verify that the system indeed boots the `shim` boot loader in the first stage. What is the full path name of this boot loader?
:::

`shim` is the first step of secure boot, and we can prove that by printing `bootorder` to this end we will be using `efibootmgr -v` command
Output is shown in the picture below

<center>

![](https://i.imgur.com/OarHy5Y.png)

</center>

Another way to see `bootorder` is to use command `bootctl status`

<center>

![](https://i.imgur.com/cCWkM57.png)

</center>

`shim` can be found by the path `/boot/efi/EFI/ubuntu/shimx64.efi`


:::info
4. Verify that the `shim` bootloader is indeed signed with the “Microsoft Corporation UEFI CA” key.
:::

In order to verify `shim` verification, i will preform several steps:

exporting keys
`mokutil --export --db`
change format of key
`openssl x509 -inform DER -outform PEM -in DB-0001.der -out DB.pem`
verify them
`sbverify --cert DB.pem /boot/efi/EFI/ubuntu/shimx64.efi`

On the last step we are getting output shown on the picture below:

<center>

![](https://i.imgur.com/ZJaAfmC.png)
Succesful verification
</center>

on the two pictures below you can see what kind of certificates are used in process of verification, and the certificate that was used to validate verification

<center>

![](https://i.imgur.com/SqF5Grx.png)
list of certificates
</center>

<center> 

![](https://i.imgur.com/6Y0Ww3e.png)
certificate details
</center>


:::info
5. What is the exact name of the part of the binary where the actual signature is stored?
:::

On the picture below you can see part of RFC2459 that describes certificate fields, and the certificate itself is stored in the `signature field`

![](https://i.imgur.com/ktled5y.png)
[RFC 2459](https://www.ietf.org/rfc/rfc2459.txt)


:::info
6. In what standard cryptographic format is the signature data stored?
:::

The structure of cryptographic keys is described in CMS, syntax of it described in RFC 5652.

:::info
7. Extract the signature data from the `shim` binary using `dd`. Add 8 bytes to the location as given in the data directory to skip over the Microsoft `WIN CERTIFICATE structure` header (see page 14 of the specification if you are interested).
:::

In extracting the signature data `pyew` - an application written in python can help us. First step of analysis can be done with `./pyew.py /boot/efi/EFI/ubuntu/shimx64.efi` command. After process of the analysis is done, our choice is to print out headers of the binary file with the command `pyew.pe.OPTIONAL_HEADER.DATA_DIRECTORY`
The result you can see in the picture below

<center>

![](https://i.imgur.com/PQcBTsB.png)

</center>

In the picture below you can see the exact part of `shim` binary where is certificate is located, the blue part is the part that needs to be skipped in order to skip over `Microsoft WIN CERTIFICAT`

[Address settings table](http://cs.usu.edu.ru/docs/pe/dirs4.html#4)

<center>

![](https://i.imgur.com/Am7fqo7.png)
Hexadecimal representation of `SECURITY` part of the `shim`
</center>

<center>

![](https://i.imgur.com/F2SZGZw.png)
Shim with the skipped 8 bytes
</center>

:::info
8. Show the subject and issuer of all X.509 certificates stored in the signature data. Draw a diagram relating these certificates to the “Microsoft Corporation UEFI CA” certificate.
:::

All certificates that are used can be found by printing `sbverify --list shimx64.efi`, the output of this program is a list, but it is better to understand them with diagram

<center>

![](https://i.imgur.com/SqF5Grx.png)
list of certificates
</center>

<center>

![](https://i.imgur.com/1FbeNGP.png)
Microsoft Corporation relating  certificates diagram
</center>

---
## Task 3: GRUB

:::info
9. Using your new knowledge about Authenticode binaries, extract the signing certificates from the GRUB boot loader, and show the subject and issuer.
:::

As in the previous steps, we can obtain information about certificates in different ways:
sbverify
<center>

![](https://i.imgur.com/GgcQciu.png)
`sbverify`
</center>

or using python application
<center>

![](https://i.imgur.com/iEEauqv.png)
headers of the binary file
</center>

<center>

![](https://i.imgur.com/UMTf7ty.png)
Hexadecimal representation of `SECURITY` part of the `grub`
</center>

so can we can say name subject and issuer of `grub`:
subject - ``/C=GB/ST=Isle of Man/O=Canonical Ltd./OU=Secure Boot/CN=Canonical Ltd. Secure Boot Signing (2017)``

issuer - `/C=GB/ST=Isle of Man/L=Douglas/O=Canonical Ltd./CN=Canonical Ltd. Master Certificate Authority`

:::info
10. Why is storing the certificate X in the shim binary secure?
:::

On the step, when we are verifying `grub`,` shim` is already verifyed and secure, so it is logicaly that we are storing certificate for the next step `grub` in the step before `shim`
I case certificate is changed inside `shim`, there will be no verefication between` MS third party marketplace` and `shim`, so we couldn start them secured

:::info
11. What do you think is the subject CommonName (CN) of this X certificate?
:::

CN of this certificate  is Canonical Ltd. Secure Boot Signing (2017)

:::info
12. Obtain the X certificate used by the shim to verify the GRUB binary
:::

We can obtain the certificate in different ways, with `mokutil --export mok0001`, or `shim-to-cert` as a part of `opencoke pkg`

MOK is an machine owner key - one of the keys is used

we can see the result of it in  the pictures below

<center>

![](https://i.imgur.com/z8kUXJQ.png)
certificate, in der and pem format
</center>

<center>

![](https://i.imgur.com/qMb5LSo.png)
weight of the certificate exactly 1080
</center>

<center>

![](https://i.imgur.com/vdGZavP.png)

</center>

:::info
13. Verify that this **Canonical Ltd. Master Certificate Authority** certificate’s corresponding private key was indeed used to sign the GRUB binary.
:::

In order to verify correspondence to grub we can analyse the binary file, or we can preform `sbverify --cert MOK-1.pem /boot/efi/EFI/ubuntu/grubx64.efi`, on order to verify it:

<center>

![](https://i.imgur.com/W9zJFK7.png)
binary file
</center>

<center>

![](https://i.imgur.com/nh7MCIG.png)
succesful verifications
</center>

---
## Task 4: The kernel

:::info
14.  Verify the kernel you booted against the **Canonical Ltd. Master Certificate Authority** certificate.
:::

We can use files like `vmlinuz`, in order to verify that kernel is signed with Canonical Ltd. Master Certificate Authority

<center>

![](https://i.imgur.com/G61cBhe.png)
succesful verifications
</center>

[Why we can use these files to verification?](https://en.wikipedia.org/wiki/Vmlinux)

:::info
15.1 **BONUS**: Where does GRUB get its trusted certificate from? Hint: It is not stored in the binary, and it is not stored on the file system.

**they are autogenerated**

15.2 **BONUS**: We have now verified Steps 1–3 of the Ubuntu chain of trust.
Verifying Step 4 is left as a bonus.
:::       
 
We can see all modules available by typing `lsmod`, then we can get more info on particular module `modinfo video`

![](https://i.imgur.com/tGcJByW.png)

and as we can see modules uses keys that was genereted by the kernel

:::info
16. Draw a diagram that shows the chain of trust from the UEFI PK key to the signed kernel. Show all certificates, binaries and signing relations involved.
:::

Every step, that have been done today, can be shown on the diagram below
![](https://i.imgur.com/OD8aVx3.png)

/proc/keys list of every cert
openssl x509 -noout -subject -in /etc/ssl/certs/ pems of certs
osslsigncode extract-signature -pem -in /boot/efi/EFI/ubuntu/grubx64.efi -out grubSign.pem


---
## References:

1. 



