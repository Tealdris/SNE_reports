###### tags: `INR`
:::success
# INR Lab 6 Assignment:  Do Whatever You Want

**Name: Radomskiy Ilia**
:::

## Task 1: Monitoring

prtg
ntopng

:::info
a. Try to do traffic monitoring on one of the networking devices from your previous labs and try to monitor and save the traffic that is going from/to a VM, This is very important in case you will have to do some networking investigation.
:::

Network monitoring is a very useful and powerful utility that can be helpful in different situations [1]
Cacti one of them
Let's install and try it out [2]

In order to work, Cacti requires several additionally installed things such like:
Apache & PHP +  Extensions
MariaDB [3]
SNMP

I will be trying to do this in the topology that is shown in the picture below.

<center>

![](https://i.imgur.com/pN7N1sT.png)
Topology of the second lab
</center>

After installation I have added nat rules on my MikroTik router, this is required because the admin's pc have private IP.

```
chain=dstnat action=netmap to-addresses=192.168.20.2 to-ports=80 protocol=tcp in-interface=ether3 dst-port=8079 
```
```
chain=dstnat action=netmap to-addresses=192.168.20.3 to-ports=80 protocol=tcp src-address=192.168.20.0/24 dst-address=192.168.122.106 in-interface=ether3 dst-port=8079
```

After that, I am able to see the web page of Cacti.

<center>

![](https://i.imgur.com/7nAjDr6.png)
Cacti main page
</center>

Admins pc is monitored now, but I gave him some action in order to see how it will be captured

<center>

![](https://i.imgur.com/3X8aMej.png)
Monitoring graphics
</center>

Left-up: Show how does pc is loads on average, there is a peak because I have loaded pc with `dd if=/dev/zero of=/dev/null`

Left-down: Memory usage, there is a peak because I have installed one application

Right-up: Login information

Right-down: Process information

There are some useful tools like Nmap and Wireshark, that was covered in previous labs. But I can quickly make and monitoring and scanning on new parts of the topology

For example, we can see new open ports that are available on pc admin by using Nmap

<center>

![](https://i.imgur.com/hCRGFJa.png)
Nmap scanning
</center>


And by using Wireshark we can locate and capture interaction session between my local machine and pc admin

<center>

![](https://i.imgur.com/Z3oNfxL.png)
Wireshark capture
</center>

Or things like snort can be used in preventing attacks and monitoring the network
 After installation, you can start snort in monitoring mode by typing this command
```
sudo snort -d -l /var/log/snort/ -h 192.168.1.0/24 -A console -c /etc/snort/snort.conf
```
In our case, we will be listening network `192.168.1.0`, printing output on the console, and saving it to log file

Snort are capturing such activitiel like

<center>

![](https://i.imgur.com/LbQgA0t.png)
Terminal output of the snort
</center>

There you can see a warning about information leak (nmap)
ICMP PING scanning warning (ping)
Misc activity (scappy packet crafting)


## Task 2: Packet crafting

I will be doing packet crafting in the topology of lab 4
I will be using scapy for this end

<center>

![](https://i.imgur.com/zK7e7Hz.png)
TOpology of 4 lab
</center>

Packet crafting is a technique that allows checking/attack/find vulnerabilities in  the networks

<center>

![](https://i.imgur.com/r24QqVg.png)
Scapy welcome page
</center>

**Simple Packet crafting:**

I have created simple IP packet and sent it to the router

<center>

![](https://i.imgur.com/tS2qsMJ.png)
IP packet
</center>

You can see them in Wireshark

<center>

![](https://i.imgur.com/18drmwp.png)
Monitoring of capturep packets
</center>

**Port scanning:**

Any port scanning happens when you send a packet to the port, and if you get any reply, then it means the port may be open

Command in scapy is look line in the picture below

<center>

![](https://i.imgur.com/wBtkmeD.png)
POrt scan packet, and reply to sending
</center>

In the answer section, you can find that we get replies for port 22 or ssh port.
You can see on the picture below, how does port scanning look like from the perspective of the router

<center>

![](https://i.imgur.com/UYf3hp6.png)
Wireshark has  captured port scanning
</center>

**Sniffering:**

By typing sniff(), we can activate sniffer mode that will capture and analyse every incoming packet.
So in captured packets, we can we pings, that was sent by me, and the ARP message from 192.168.1.1

<center>

![](https://i.imgur.com/aHmPgEX.png)
Comming packets
</center>


**TCP connection:**

TCP connection (TCP three-way handshake) consists of 3 parts

Sending SYN on both sides
Receiving SYN on both sides
Sending and receiving ACK 

Every part of this `three-way handshake` will be recreated in the scapy

In the picture below you can see how does three-way handshake in terminal

<center>

![](https://i.imgur.com/0KykmrE.png)
Scapy terminal
</center>

In wireshark you can see how does es

<center>

![](https://i.imgur.com/fwBDBEB.png)
Three-way handshake in wireshark
</center>

**ospf interception:**

In this part, I will advertise an unexisting router machine as an OSPF router. This method of attack maybe use in order to intercept traffic.

I set up settings that are listed below

<center>


![](https://i.imgur.com/KCnFNJ1.png)
![](https://i.imgur.com/8Cd1xEL.png)
![](https://i.imgur.com/TLIsvC2.png)
Settings that will be used in crafting
</center>


<center>

![](https://i.imgur.com/zbsan3L.png)
OSPF adverizement of 47.47 router
</center>



---
## References

1. [Best Free Network Monitoring Software](https://reviews.gns3.com/best-free-network-monitoring-software/)
2. [Install Cacti On Ubuntu 20.04](https://www.itzgeek.com/post/how-to-install-cacti-on-ubuntu-20-04/)
3. [Как установить MariaDB 10.4](https://infoit.com.ua/databases/mariadb/kak-ustanovit-mariadb-10-4-v-ubuntu-18-04-ubuntu-16-04/)
4. [Squid на Ubuntu](https://www.dmosk.ru/instruktions.php?object=squid-ubuntu)