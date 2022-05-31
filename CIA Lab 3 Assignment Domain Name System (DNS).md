###### tags: `CIA`
:::success
# CIA Lab 3 Assignment: Domain Name System (DNS)
**Name: Radomskiy Ilia**
**Variant: 10**
:::

---
## Task 1: Downloading and Installing a Caching Name Server

Because of my 10'ts number, I will be using BIND9 for the DNS deployment.


:::info
1.1 Validating the Download
:::

➢ Why is it wise to verify your download?

There were some cases of breaking in the files, changing them and leaving back some viruses or backdoor
Because of that aware user should check things that he is downloading
One of the tools that may be used to this end is hashing.

Hash is a convolution process, that creates information in a file into one string, it uses a special algorithm.
In the end, there is a checksum that will be verification of the correct, unmodified file.

Digital signature - mathematical scheme. It uses for verification. 


➢ Download the BIND tarball (also if you are doing the Unbound+NSD part) and 
check its validity using one of the signatures.

Now I checked the downloaded file, in order to do that I downloaded public signature from the

```
wget https://ftp.isc.org/isc/bind9/9.11.31/bind-9.11.31.tar.gz.sha256.asc
```

Result of the compearing with the signature of the downloaded file is shown in the figure 1

<center>

![](https://i.imgur.com/XtS1iVT.png)
Figure 1: Comparing two signatures, one from BIND another  signature from the website
</center>

➢ Which mechanism is the best one to use (signatures or hashes)? Why?

Algorithms of creating signatures are slower than hashes, but have several positive sides: the receiver is required only for the public key, the file is more secure if there is no exchange of the private keys

On the other hand, hash functions are hard to reverse functions, and they don't contain any information to prove contained, but they are fast

So Digital Signature is the better in terms of the information and the content validation [1]

:::info
1.2 - Documentation & Compiling
:::

In order to install an application user might use ```apt install```, or, like in our case, configure and install application manually

First, I downloaded the file, unarchived it, and install several dependencies

```
wget https://downloads.isc.org/isc/bind9/9.16.0/bind-9.16.0.tar.xz
tar -xf bind-9.16.0.tar.xz
sudo apt install build-essential pkg-config python3-pip libuv1 libuv1-dev libssl-dev libcap-dev libtool-bin
libnghttp2-dev
libjemalloc-dev

```

I wrote

```
./configure --localstatedir=/var --sysconfdir=/usr/local/etc/bind
```

line in order to install BIND
after this is configured I send two commands sequentially
```
make
make install
```
in order to install utility

directory ```/var``` will be used as storage for the information of bind

in this place configuration files will be stored ```/usr/local/etc/bind```

But before the process of installation, I checked possible previously installed BIND applications by printing
```
named -v
```

➢ What is the difference between/etc ,/usr/etc ,/usr/local/etc

/etc - used as storage of the system services configuration files
/usr/etc - used as storage of the program configuration files
/usr/local/etc - used as storage for configuration files of the manually installed programs [2]

---
## Task 2: Configuring Caching Name Server

➢ Why are caching-only name servers still useful? 

Because it is easier to store information in local memory, and not ask for the IP address every time. For example, some of the home routers save some of the DNS requests

caching-nameservers allow to resolve private zones

:::info
Root Servers
:::

The root server - is the most up-top server that contains information about zones below him. So if you need to resolve an unknown domain name first server you ask is the root server. Usualy it refered ad ```.```

For establishing your own DNS server you need a configuration file with the IP addresses of the root servers

It is located in 
```/usr/local/etc/bind/named.root```

It might be downloaded from

```ftp://ftp.rs.internic.net/domain```

:::info
Resolving localhost
:::

Information about localhost store near to the ```named.root```
Config file called ```named.local``` 
Localhost is a reserved domain name for the 127.0.0.1, or IP for PC in a local network environment
The Configuration file of the ```named.local``` you may see in figure 2

<center>

![](https://i.imgur.com/X1fBS5G.png)
Figure 2: ```named.local``` Config file
</center>

This file contains in it some of the records that needs to be translated
A - IPv4 record
AAAA - IPv6 record
NS - name server
SOA - start of Authority (tells where record of the domain contains)
IN - internet

:::info
Access control
:::

In order to set up Access control on our DNS server we write the access control list in the ```named.conf``` file. COnfiguration of the file is in figure 3


<center>

![](https://i.imgur.com/XbxCFqt.png)

Figure 3: ```named.conf```  file
</center>

Line ```acl testnet { 172.31.8.74/32; localhost; };``` Requires for the access control functions, in our case I allowed to enter DNS from the local network

➢  Why access control is important for a recursive servers?

recursive servers have a high possibility of being attacked via cash poisoning, reflection or amplification attack
That is why it is important to give access only to the trusted clients

:::info
Testing
:::

Before DNS server deployment it is important to check every config you have build
I performed it with the
```
named-checkconf
```
and
```
named-checkzone 127.0.0.1 /usr/local/etc/bind/named.local
```
commands. 
In figure 4 result of the checking, process may be seen.

<center>

![](https://i.imgur.com/MIqKURI.png)

Figure 4: Checking config files
</center>

The empty output of the ```[named-checkconf](https://maxblogs.ru/articles/dns-server-bind#subzone)``` means that everything is correct.

➢   Why do the programs return a result value?

Because the main purpose of any kind of program is to change or calculate something, and, simply because humans must understand what just happened, programs return any kind of value. Result value may be different: a calculated number, error code or a little bit more information about the process of the program itself [3]

---
## Task 3: Running Caching Name Server

RNDC translates as Remote Name Daemon Control, and it is used, as it stands in the name, for remote access to the DNS server, in our case [4]
I used the ```named``` command in order to start my BIND9-DNS server. But you might not always have access to the server in a physical sense, so it is very important to be able to manipulate it online. To this end, I used ```rndc``` server control.

➢ Show the changes you made to your configuration to allow remote control 

First thing configure file with the key in it, it also contains additional information, "default-server" for example
Those generated keys will be used in order to get remote access to the DNS server[5]

<center>

![](https://i.imgur.com/GOOVkMt.png)
![](https://i.imgur.com/iaZh3AE.png)
Figure 5: ```rndc.config``` and ```rndc.key``` files
</center>

After the file is configured it is also important to tell our main configuration file that those are created. I performed that by adding lines in the ```named.conf```

<center>

![](https://i.imgur.com/HxnHCXg.png)

Figure 6: ```named.conf``` file
</center>

Extra line control means that we allow remote access to the server from ```10.1.1.67``` IP address

From this point, we can easily access the DNS server from our local machine (figure 7)

<center>

![](https://i.imgur.com/fxKuadY.png)

Figure 7: Remote DNS server operating
</center>

It is also important to log the behaviour of our server, so, when the error comes, you are will be able to find out the cause of the error and fix it [6]

In order to do that I added several lines in the ```named.conf``` file, and, of course, have created a directory, where log information will be stored

<center>

![](https://i.imgur.com/aievLwx.png)
Figure 8: Lines that allow logging
</center>

lines of the config line mean
```severity notice;``` - what kind of actions will be recorded
```print-... yes;``` - saves specified parameter to the log file
```category ...``` - category of logging information


➢ What other commands/functions does rndc/unbound-control provide?

Besides basic commands, discussed previously, ```rndc``` has a lot of additional control options. The user basically can operate DNS server remotely, all local functionality is available to the online user, for example:
addzone/delzone
flush[/view/name/tree]
re config/cursing/frshing/loading/transfering
a lot of signing options
etc

➢ What is the difference between stop -> start and reload?

The main difference between start&stop and reload is that is ```reload``` command does not delete cash, while ```start&stop```  does

➢ What do you need to put in resolv.conf (and/or other files) to use your own name server? 

In terms of the local PC, the resolver is a small DNS user, it works with the DNS request from applications. The is located on the ```127.0.0.53``` IP address
In order to configure the local resolver, we change some lines by printing ```sudo nano /etc/systemd/resolved.conf```
I have added ```dns = 127.0.0.1```
After restarted ```systemd-resolved```

<center>

![](https://i.imgur.com/pTTGVDX.png)
Figure 9: ```resolved.conf``` file
</center>

From this point, our default DNS resolver is configured on the local machine(figure 10)

<center>

![](https://i.imgur.com/cESxjJk.png)
Figure 10: ```systemd-resolve --status``` output
</center>

In figure 11 you can see how resolved DNS name of ```google.com``` comes from the local resolver

<center>

![](https://i.imgur.com/lxm10yO.png)
Figure 11: Reaching ```google.com```
</center>

---
## Task 4: Authoritative Name Server

➢ What is a private DNS zone? Is st10.sne21.ru private? 

Private DNS - is a DNS that is accessible only for the users of the virtual private network, and it resolves only local requests
st10.sne21.ru is a private DNS zone because I couldn't find it from outside, and my TA said that this domain is
"inside your VPN network"
On the other hand, ns record of the server sne21.ru may be foumd by using dig command

➢ What information was needed by TAs so they can implement the delegation?

In order to delegate DNS zone, the DSN record file must contain this information:
NS and A records that will point on servers inside the zone, that information contains in it a name of the zone and IP address of the server

➢ 2 MX records. Make sure that mail for your domain is delivered to your own 
public IP. We will use the MX records later on.
➢ 4 A or AAAA records. Use your imagination.
➢ 2 CNAME records
➢ Show the resulting zone file.

<center>

![](https://i.imgur.com/h6nPWRC.png)
Figure 12: zone configuration file
</center>

I added in my ```st10.sne21.ru``` several lines, that tells my DNS how to work, for example:
```
IN MX 10 mail.st10.sne21.ru. 
IN MX 10 mail.st10.sne21.ru.
```
Means that I have 2 mail servers in the zone

```
www IN CNAME st10.sne21.ru.
vvv IN CNAME bing.com.
```

Means that DNS server redirect ```www``` and ```vvv``` requests to my zone ```st10.sne21.ru.``` and ```bing.com``` name respectively

```
ns      IN      A       10.1.1.67
ns2     IN      A       10.1.1.81
mx      IN      A       10.1.1.67
mail    IN      A       10.1.1.68
```
all A records describe the IP of the services that the user need to be redirected from the DNS name

➢ Add a reference to your zone file to named.conf or nsd.conf, respectively.
➢ Restart or reload the name server and test your configuration using the provided 
tools(Hint: dig, drill). 

After the config is saved and the server is reloaded, I checked the availability of the records I have made. Everything is check by the command ```dif <domain>```. All proof of work may be seen in figures 13, 14, 15

<center>

![](https://i.imgur.com/W9i4Txi.png)
![](https://i.imgur.com/6CO64h9.png)
FIgure 13: CNAME availability
</center>

<center>

![](https://i.imgur.com/huS1qHH.png)
Figure 14: Mail services
</center>

<center>

![](https://i.imgur.com/Cn9TE1a.png)
Figure 15: NS section
</center>


[9]

## Task 5: Delegating Your Own Zone (BONUS)

➢ How did you update ```st<X>.sne21.ru.``` zone file to delegate a subdomain to your partner?

In order to delegate subdomain zone to someone, st12 in my case, I need to change  config file:
```db.st10.sne21.ru```, shown in figure 16, the most important lines file is:
```
$ORIGIN st12.st10.sne21.ru.
@   IN NS ns3.st12.st10.sne21.ru.
ns3 IN A 10.1.1.67
```

```$ORIGIN st12.st10.sne21.ru. ``` - this line stands for the creation of a new zone

```@   IN NS ns3.st12.st10.sne21.ru.``` - set name server for the domain name

```ns3 IN A 10.1.1.67``` - gives an IP  to the name server

There may be some additional lines like ```$TTL``` and etc, (like in lines below than st12 sub-zone, they have been created in order to give sub-zone to the virtual machine)
 
<center>

![](https://i.imgur.com/lIr5xDB.png)
Figure 16: zone configuration file
</center>

➢ Show content of ```st<X>.st<Y>.sne21.ru``` zone file that you have 
delegation for.

The next thing to manage is the new zone file. This file is used for BIND9 to know about the new zone, that the DNS server might be working with. They're also required to markdown ```named.conf``` in order to connect the main configuration file and zone file (figure 18)

In the case of the new ```st10.st12.sne21.ru``` file, the lines listed below means:
```IN NS ns3.st10.st12.sne21.ru.``` - set name server for the domain name
```ns3 IN A 10.1.1.212``` - gives to the ns IP address

<center> 

![](https://i.imgur.com/wBFbM8I.png)
Figure 17: ```st10.st12.sne21.ru``` zone configuration, ```master.com``` in my case
</center>

In the case of the configuration of my partner, it is the same, but reversed in the case of IP's:
My file - IP of groupmate ```10.1.1.212```
Grupmate's file - my IP ```10.1.1.64```

<center>

![](https://i.imgur.com/HwlD8RR.jpg)
Figure 18: ```st12.st10.sne21.ru``` zone configuration, ```master.db``` in case of my groupmate
</center>

➢ What named.conf options did you add or change?

```named.config``` file changed to correspond to zone configuration settings
Brand new zone ```st10.st12.sne21.ru``` is a zone that has been created by my partner, and this configuration contains itself a ling to the ```master.com``` file, aka ```st10.st12.sne21.ru.```
 
<center>

![](https://i.imgur.com/E3hSIUZ.png)
Figure 19: Additional lines in the ```named.conf```
</center>

<center>

![](https://i.imgur.com/cMDRNmi.jpg)
Figure 20: Additional lines in the ```named.conf``` in case of my groupmate's configuration file
</center>

in order to give control over my subdomain I wrote slightly different lines:

```
zone "st12.st10.sne21.ru" IN {
     type forward;
     forwarders { 10.1.1.212; };
};
```
They are means that I give control over ```st12.st10.sne21.ru``` to the users with specific IP ```forwarders```


➢ Show the results of the tests that you performed. 

Let's make some tests in order to prove that everything, that has been set up before, is working.

1 My domain ```st10.sne21.ru```

In task 4 it already has beep shown proof of work of the DNS server, in figure 21 there is an additional photo, with the usage of the ```drill```

<center>

![](https://i.imgur.com/3HvZVYy.png)
Figure 21: ```drill``` to the ```st10.sne21.ru```
</center>

2 My subdomain ```st12.st10.sne21.ru``` and ```st105.st10.sne21.ru```

Created by me subdomain ```st12.st10.sne21.ru``` is handled by my groupmate, on figure 22 you can see how does ```dig``` command works with his domain

<center>

![](https://i.imgur.com/Ai6NAVe.png)
Figure 22: domain ```st12.st10.sne21.ru``` delegated to my groupmate
</center>

The result is an SOA record, that tells the name server of the domain

Additionally, I have made a VM with the installed BIND9 on it, then I have created several records, that delegated sub-domain ```st105.st10.sne21.ru``` to VM, the same process as for creating sub-domain to my partner. The result of the ```dig``` command you may see in figure 23.

<center>

![](https://i.imgur.com/h7AlY2G.png)
Figure 23: domain ```st105.st10.sne21.ru``` delegated to my VM
</center>

The result of the command ```dig``` some kind of familiar to the previous one (figure 22), it is an SOA record

3 Domain of my groupmate ```st12.sne21.ru```

Before domain delegation, let's check if the domain of my groupmate is available at all.

<center>

![](https://i.imgur.com/nC2lYhD.png)
Figure 24: ```st12.sne21.ru``` availability
</center>


<center>

![](https://i.imgur.com/krjfuLr.png)
Figure 25: ```mx.st12.sne21.ru``` availability
</center>

<center>

![](https://i.imgur.com/6uBxQ0j.png)
Figure 26: ```host1.st12.sne21.ru``` availability
</center>

as you can see from the pictures (24, 25, 26), some of the services have records, which means DNS is working

4 sub domain nof my groupmate ```st10.st12.sne21.ru```

At this stage, my partner has created a subdomain and have deligated it to me.
In figure 27 you may see the configuration file of the subdomain, I can change it in order to set up services, mail and etc.

<center>

![](https://i.imgur.com/o5KovEP.png)
Figure 27: File of the deligated zone
</center>

digging to the ```st10.st12.sne21.ru``` gives as an SOA record, which means I have access to the delegated domain.

<center>

![](https://i.imgur.com/HKrQPsV.png)
Figure 28:
</center>

It is also available and applied on the VM, the result you can see in picture 29
<center>

![](https://i.imgur.com/pvaNBth.png)
Figure 29: ```st105``` sub-domain on the VM
</center>


So that is how deligation of the sub-domain names works[10]

## Task 6: Zone Transfers (BONUS)

➢ How did you set up the secondary nameserver? Show the changes to the configuration files that you made.

➢ What happens if the primary name server for the subdomain fails?

It depends on the type of server, for example, if the slave server saves information about the domain zone in it, that server might take responsibility of the master server. The forward server just redirects the request to another server. In the case of a cache-only server cant answer all requests, but it can still give a remembered addresses. 
So in order to prevent the down state of the server DNS is used in pairs, the ore in more amount of server state. That is standard now.

➢ Show the changes you had to make to your configuration
➢ Use a DNS tool (for instancedig ) to initiate a zone transfer from your primary 
name server to your secondary.

➢ Show the output of the DNS tool and describe the steps in the transfer process.
➢ What information did the secondary server receive?



1: [Checking the GPG Signature](https://goo.su/7Gkl)
2: [BIND 9 Administrator Reference Manual](https://bind9.readthedocs.io/en/latest/index.html)
3:[Installing a cashing name server using bind 9](https://medium.com/@chrixsbassey/downloading-and-installing-a-caching-nameserver-using-bind-9-from-source-75b555bc7e10)
4: [Configure RNDC Key for Bind9](https://tecadmin.net/configure-rndc-for-bind9/)
5: [Configuring rndc](https://subhrajitnandy.wordpress.com/configuring-rndc/)
6: [Настройка логов Bind9](https://ixnfo.com/bind9-logging.html)
7: [Set Up Local DNS Resolver](https://www.linuxbabe.com/ubuntu/set-up-local-dns-resolver-ubuntu-20-04-bind9)
8: [Setting up a DNS zone with Bind9](https://www.debuntu.org/how-to-setting-up-a-dns-zone-with-bind9/)
9: [HOWTO DNS сервер BIND](https://www.k-max.name/linux/howto-dns-server-bind/)
10: [DNS сервер BIND](https://maxblogs.ru/articles/dns-server-bind#subzone)