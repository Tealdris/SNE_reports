###### tags: `INR`
:::success
# INR Lab 2 - IPv4 & IPv6
**Name: Radomskiy Ilia**
:::




## Task 1: Ports and Protocols 
:::info
1 Check the open ports and listening Unix sockets against ssh and HTTP on Admin and Web respectively.
:::

 In order to check what ports that are open we will use "netstat", "lsof" utilities[1][2]

Let's start with "web" PC
For "netstat" command we will be using those keys "-tulpn". They actually mean:
**t** for TCP ports
**u** for UDP ports
**l** for listening
**p** for the name of the program
**n** for show IP in numeric

<center>
 
![](https://i.imgur.com/JXpdt5i.png)
 Figure 1: open ports on "web" PC via "netstat" utility
</center>
 
For "lsof" command we will be using those keys "-i -P". 
hey actually means:
**i** for open connections to show
**P** for ports to show


<center>

![](https://i.imgur.com/C3FF2t4.png)

Figure 2: open ports on "web" PC via "lsof" utility
</center>

Some of the utilities that may help us to list open ports are:  "zenmap", etc. But we will not be covering them.

For "admin" PC

<center>

![](https://i.imgur.com/jQjx75z.png)
Figure 3: open ports on "admin" PC via "tulpn" utility
</center>

<center>

![](https://i.imgur.com/aGB8RV7.png)
Figure 4: open ports on "admin" PC via "lsof" utility
</center>


:::info
2 Scan your gateway from the outside. What are the known open ports?
:::
We can use "Nmap" utility in order to find what is possible running ports on the machine. To this end we print 
```
nmap 192.168.122.106
```
On the picture we can see that on our gateway ports available is:
21, 22, 23, 80, 2000, 8080, 8081, 8082, 8083, 8291, where is 8080, 8081, 8082, 8083, if ports that was set up in previous lab

 <center>

![](https://i.imgur.com/YqrFPYX.png)
Figure 5: Scaning gateway from our workstation
</center>


:::info
3 A gateway has to be transparent, you should not see any port that is not specifically forwarded. Adjust your firewall rules to make this happen. Disable any unnecessary services and scan again.
:::

Some ports are listening by the pre-configured settings, so you might want to turn them off
Ways of doing it can vary from one router to another [3]
You can easily do this for Mikrotik from the web interface figure 6

 <center>

![](https://i.imgur.com/hI5TXPn.png)
Figure 6: Disabling port 2000
</center>

Or from the command line
figure 7

<center>
 
![](https://i.imgur.com/bIT00DN.png)
Figure 7: Disabling ports 21 and 8729
</center>

Or use filter ob the port like port 22 drop

After we closed every port that we don't need, we can see only ports that were configured in the previous lab figure 8.

<center>
 
![](https://i.imgur.com/TPfg9Iq.png)
Figure 8: Nmap scanning after several ports were disabled
</center>

:::info
4 It supposes that some scanners start by scanning the known ports and pinging a host to see if it is alive
:::

4.1 Scan the Worker VM from Admin. Can you see any ports?

Let's scan the worker's PC from the admin's by using Nmap. You can see the results in figure 9

<center>

![](https://i.imgur.com/82coDJ4.png)
Figure 9: Nmap scanning from admin to  worker results 
</center>

Figure 9 means that the worker's PC is running ssh server and we can access to it via ssh

<center>

![](https://i.imgur.com/UJR0K7E.png)
Figure 10: Getting access to the worker via ssh
</center>

4.2 Block ICMP traffic on Worker and change the port for SSH to one that is above 10000.

We can make our system a little bit secure by changing ssh port from standard 22 to unusual 20010. In figure 11 you may see that we can get access to the worker machine via SSH and a new port for it

```
sudo vi /etc/ssh/sshd_config
```
<center>

![](https://i.imgur.com/CYfwpx6.png)
Figure 11: Reaching workers PC
</center>

We also can block ICMP traffic on the machine to make your machine less noticeable
```
iptables -A INPUT --proto icmp -j DROP
```

<center>

![](https://i.imgur.com/fp2xPVi.png)
Figure 12: Iptables with new rule that blocks ICMP
</center>


4.3 Scan it without extra arguments.

When we use default nmap scanning parameters we might miss some information about the target. As figure 13 shows, we missed ssh port
<center>

![](https://i.imgur.com/siyCnqu.png)
Figure 13: Scaning for open ports
</center>

4.4 Now make necessary changes to the command to force the scan on all possible ports

In order to see missed port, we change Nmap command[4]
```
nmap -p0-30000 192.168.30.2
```

This command stands for a scan every port from 0 to 30000 on target 192.168.30.2

<center>

![](https://i.imgur.com/uHHNzem.png)
Figure 14: Nmap command that actually found our machine
</center>


4.5 Gather some information about your open ports on the Web (ssh  and HTTP )

So, every service that is currently working on your PC may be seen by users of the network
For example, "web" PC has more ports than the admin, you can see than on the figure 15, its simply because we have web server on the web machine

<center>

![](https://i.imgur.com/8xvFyuK.png)
Figure 15
</center>

Or make nmap scanning everything 

`nmap -sS -sV -sC -n -v 192.168.20.2`

---
## Task 2: Traffic Captures & IPv6 
:::info
1 Access your Web Page from the outside and capture the traffic between the gateway and the 
bridged interface.
:::

Now we try to access the webserver on 192.168.20.2 from our work station, at the same time we will be recording traffic, that will be sent between both PCs. You can see the process of capturing in figure 16

<center>

![](https://i.imgur.com/ZlVK9Lj.png)
Figure 16: Traffic that was captured
</center>

In this case, we can see several lines of transmission:
ARP (address resolution protocol) - this protocol find's out the MAC address of the network used from the IP
NTP (network time protocol) protocol that allows synchronizing the internal clock of the machine
TCP (transport control protocol) is one of the main protocols of transmission on the internet, in our case we connect from our workstation port "46372" to the port of the MikroTik "8080". 

After connection establishment, our machine asks for "GET" inquiry from the webserver
In our case server sends back HTTP traffic to the workstation.
In a dump there are initial lines of "keep-alive", this protocol helps to not to close the connection if it is working at the moment

Headers of the packages may contain some useful information, for example, we can know that on the machine "Nginx" server is working. We also can see what kind of OS is working on the server

![](https://i.imgur.com/8JNLAMw.png)


:::info
2 SSH to the Admin from the outside and capture the traffic (make sure to start capturing 
before connecting to the server).
:::

Now we will try to connect to the admin PC from our workstation, at the same time we will be capturing the traffic that will come through the network. Figure 17.
In Wireshark there are several built-in functions, one of them is the ability to decode TCP packets, in our case we may specify SSH message, and make capturing more readable

<center>

![](https://i.imgur.com/lNSF9m3.png)
![](https://i.imgur.com/8C11gO5.png)

Figure 17: Wireshark and terminal windows.
</center>

In the terminal window, you may see the process of getting access to the address.
On the Wireshark window, you may see serval interactions between the admin and our workstation.
For example, you may see key exchange processes and possible ways of ciphering like 
Key exchange chipping: Diffie-Helman, or other possible ways of encryption like 'chacha20' or 'aes128'

The process of ssh connection is:
1 TCP connection establishing
2 Exchange of ssh versions and choosing of algorithm process.
3 The choosing of the algorithm is a process of sending each other lists of supported algorithms
4 Obtaining a session encryption key
5 Now the process of authentication starts, when the user need to use the password
6 From this point we can send information and commands to the server

:::info
3 Configure Burp Suite as a proxy on your machine and intercept your HTTP traffic.
:::

In order to configure Burp as a proxy, we need to change several settings. And use the same settings inside Burp[5]
We need to set up proxy settings in order to intercept data

<center>

![](https://i.imgur.com/BkxPzOs.png)
![](https://i.imgur.com/CTBNC37.png)

Figure 18: Settings that was changed
</center>


From this point, we will capture  traffic from our web page on the web PC

<center>

![](https://i.imgur.com/hmkjcen.png)
Figure 19: Captured traffic
</center>

In figure 20, you may see interception to the web page.
At this point web page still loading because the package is stopped by the burp suite. And we can actually change all of the intercepting information

<center>

![](https://i.imgur.com/d3qMp8N.png)
Figure 20: Burp Suite interception 
</center>

Let's try to change some values un burp suite. We may simply delete part of the lines listed in the burp suite.

<center>

![](https://i.imgur.com/Lc9wTMq.png)
Figure 21: Process of deleting values and web page that is broken
</center>

On the other hand, you can't do the same process, of the changing packet, to the ssh packet because SSH includes a hash function in it. The second thing is encryption of the packages in ssh connection. It also prevents if the interception

There are actually several analogues to the burp suite:
Fidler
Mitmproxy
Proxyman
etc

:::info
4 Configure IPv6  from the Web Server to the Worker. This includes IPs on the servers and the default gateways.
:::

Let's configure IPv6 in our network. To this end, we change 50-cloud-init.yaml file in the PC's, like it is shown in figure 22

<center>

![](https://i.imgur.com/Z0Q6uD8.png)

Figure 22: Configuration of the webserver
</center>

Our gateway also needs to be configured. To this end, we add IPv6 addresses to the interfaces with the lines

```
ipv6  address add address=fc02:ca09:2bc5:a2f1::1/64 interface=ether1 advertise=yes
ipv6  address add address=fc01:28db:30cf:180c::1/64 interface=ether1 advertise=yes
```

<center>

![](https://i.imgur.com/Gabhbkg.png)
Figure 23: Configuration on the mikrotik
</center>

After everything is configured we may check the availability of our network by pinging. You can see how everything is working in figure 24

<center> 

![](https://i.imgur.com/mOV9O7f.png)
Figure 24: Pinging from web to admin
</center>



:::info
5 Access the Web Page using IPv6  from Admin while capturing again. Can you see the 
difference? What's the difference?
:::

After we find out that our network is working. We will try to access the web from the admin's PC.

We will be using curl command

```
curl -g -6 'http://[fc02:ca09:2bc5:a2f1::2]:80/'
```

<center>

![](https://i.imgur.com/Ai2BFrb.png)
Figure 25: Wireshark captured connection between admin and web
</center>

---
## References:

1: [netstat man](https://www.opennet.ru/man.shtml?topic=netstat&category=8&russian=2)
2: [lsof man](https://www.opennet.ru/man.shtml?topic=lsof&category=8&russian=2)
3: [MikroTik:Закрываем любые порты](https://hamsterden.ru/close-ports-on-mikrotik/#zakrivaem_54_port)
4: [Nmap man](https://nmap.org/man/ru/)
5: [Use Burp Proxy to Intercept HTTP Traffic](https://cyberpersons.com/2016/12/16/use-burp-proxy-intercept-http-traffic/)
