###### tags: `INR`
:::success
# INR Lab 3 - VLANs & Fault Tolerance
**Name: Radomskiy Ilia**
:::

---
## Task 1: VLANs 

:::info
1 Change the topology of your network to as follows, make the necessary configs
:::

I used topology from the previous lab and expanded it with the new parts and new switches.

<center>

![](https://i.imgur.com/vdFnG3Z.png)
Figure 1: New topology
</center>

<center>

![](https://i.imgur.com/KeGvW3B.png)
Figure 2: Pinging from the web to admin
</center>

<center>

![](https://i.imgur.com/kb3vsdw.png)
Figure 3: Pinging from web to google's DNS server
</center>

:::info
2 Exchange the default  switches Cumulus VX instances
:::

So instead of the basic, built-in GNS3 switches, I will be using Cumulus VX.
I install the new switches the same as I did with the MikroTik in the previous lab 

<center>

![](https://i.imgur.com/tWDChPy.png)
Figure 4: Choosing of the new cumulus switches

</center>

:::info
3 Configure the switches and make sure you have connectivity between the hosts
:::

Below there is the list of commands that I write in the Cumulus in order to have a connection between ```web```, ```admin``` and the internet.

Settings in the external switch
```clike=
interface swp1
  bridge-access 20

interface swp2
  bridge-access 20

interface swp4
  bridge-access 20

interface swp5
  bridge-access 20

interface swp6
  bridge-access 20

interface bridge
  bridge-ports swp1 swp2 swp4 swp5 swp6
  bridge-pvid 1
  bridge-vids 20
  bridge-vlan-aware yes

interface mgmt
  address 127.0.0.1/8
  address ::1/128
  vrf-table auto

interface vlan20
  address 192.168.20.254/24
  vlan-id 20
  vlan-raw-device bridge
```

:::info
4 Questions
:::


1 How do VLANs work at a packet level ?

Vlan translates for a virtual local area network. VLANs allow to put a tag on the traffic, for the network to know about the place packet should go.
It may be used as a security setting (not allow communication between pc connected to the same switch), or convenience purpose (allow to communication between two pc that connected to the different switches)

<center> 

![](https://i.imgur.com/uYz5wqV.png)
![](https://i.imgur.com/5YpvMKw.png)
Figure 5: VLAN tagging in pictogram and in the Wireshark
</center>

2 What are the two major protocols used for this ?

The two most popular protocols in VLANs is a DTP(dynamic trunking protocol) and VTP (VLAN trunking protocol)[1]
DPT - automatically creates a connection between two stitches  in trunk mode, then they are agreed on the parameters of the trunk connection
VTP - in this protocol information about VLANs available in the interconnected switch (database synchronization)
Or
802.1q
And cisco one

3 What do we mean by Native VLAN ?

Native VLANs means that all traffic that will be transmitted through this VLAN is untagged, without belonging to any VLAN

:::info
5 Configure the VLANs on the switches to isolate the two virtual networks shown below
:::

In order to isolate two networks, I put up the configuration on the 3 switches, like it is listed below [2]
You can do this in two different ways:
Changing config file locate in
```
/etc/network/interfaces
```

Adding everything line by line
```
net add bridge bridge ports swp1-6
...
```

Settings  on the administration switch
```c
interface swp1
  bridge-vids 20 30

interface swp5
  bridge-access 20

interface swp6
  bridge-access 30

interface bridge
  bridge-ports swp5 swp6 swp1
  bridge-vids 20 30
  bridge-vlan-aware yes
  bridge-pvid 1

interface mgmt
  address 127.0.0.1/8
  address ::1/128
  vrf-table auto

interface vlan20
  address 192.168.20.253/24
  vlan-id 20
  vlan-raw-device bridge

interface vlan30
  address 192.168.30.253/24
  vlan-id 30
  vlan-raw-device bridge
```

Setiings on the itdepartament switch
```c
interface swp1
  bridge-vids 30

interface swp6
  bridge-access 30

interface bridge
  bridge-ports swp6 swp1
  bridge-vids 30
  bridge-vlan-aware yes

interface mgmt
  address 127.0.0.1/8
  address ::1/128
  vrf-table auto

interface vlan30
  address 192.168.30.254/24
  vlan-id 30
  vlan-raw-device bridge
```

Setiings on the internal switch

```c
interface swp1
  bridge-vids 20 30

interface swp5
  bridge-vids 20 30

interface swp6
  bridge-vids 30

interface bridge
  bridge-ports swp5 swp6 swp1
  bridge-vids 20 30
  bridge-vlan-aware yes

interface vlan20
  address 192.168.20.252/24
  vlan-id 20
  vlan-raw-device bridge

interface vlan30
  address 192.168.30.252/24
  vlan-id 30
  vlan-raw-device bridge
```

:::info
6 Checking
:::

Let's check how does everything work
According to the setting< from this point, we should have access to HR (VLAN 20) from ITdepartment and manager (VLAN 30)

1 Ping between ITManager and HR, do you have replies?

<center>

![](https://i.imgur.com/tTtBahx.png)
Figure 5: manager pinging HR and ITmanager
</center>

2 Ping between ITManager and Management , do you have replies ?

<center>

![](https://i.imgur.com/TOU2Dy7.png)
Figure 6: ITmanager pinging HR and manager
</center>

So, as shown on the screenshots, the manager and ITmanager are separated in the VLAN 30 and have connections between them. But the HR, does not have connections to anyone because of its own VLAN 20

3 Capture the traffic  of the last ping and show in the packet the VLANs indication.

We can use Wireshark in a deeper investigation of the VLANs
In figure 7 you may see (blu marked line) how does ping look like, and that it has a VLAN ID in it

<center>

![](https://i.imgur.com/ZMzlS21.png)
Figure 7: Ping packet in the Wireshark

</center>

:::info
7 Configure Inter-VLAN Routing between Management VLAN and HR VLAN .
:::

Sometimes we do need traffic between different networks, so in order to allow connections between management VLAN and HR VLAN I change the configuration of the internal switch
I simply have not specified VLANs on the ports, so the switch does not operate with them as VLAN packets

```c
auto bridge
iface bridge
    bridge-ports swp5 swp6 swp1
    bridge-vids 20 30
    bridge-vlan-aware yes
    bridge-pvid 1
```

:::info
8 Show that you can now ping between them.
:::

On the figure, you can see how does the ping connection establishment between HR and ITmanager, and the IP address of the HS's PC

<center>

![](https://i.imgur.com/Se5lzAs.png)
Figure 8: Ping command and ip address from the HR's terminal
</center>

:::info
9 Capture the traffic going to and out of the router and show the different traffic of the sub-interfaces.
:::

We can capture traffic that comes from the HR to the manager

<center>

![](https://i.imgur.com/m62G1GL.png)
Figure 9: Wire shark output
</center>

---

## Task 2: Fault Tolerance 

:::info
1 Questions
:::

1 What is Link Agregation ?

Link Aggregation - Technology that allows us to use connections as parallel (count two connections as one), in order to prevent downtime, if one of the physical connections is out.
It also can be used in order to increase bandpass.

2 How does it work (breifly.) ?

2.1 physically connect switches
2.2 set up parameters identically
2.3 link them up (for example by creating a bond interface)

In reality, if the transferred information is too big, then it does not separates into the different links (to prevent out-of-the order)
But if the information is big enough different flows use hashing algorithms, in order to send information

3 What are the possible configuration modes ?

LACP - link aggregation control protocol it is when several parallel links to transmissions in connection into one logical

PAgP - port aggregation protocol it is when several physical interfaces is bridged in one logical

:::info
2 Use link aggregation between Web and the Gateway so that you have Load Balancing and Fault
Tolerance.
:::

In order to set up aggregation, I changed some parameters on the PC and Mikrotik
I added connection information to the second internet interface on the web pc

<center>

![](https://i.imgur.com/kfwhnpG.png)
Figure 10: Settings on the ```web```
</center>

On the Mikrotik gateway, I have added one interface and add a new bonding interface.

<center>

![](https://i.imgur.com/vohzb6c.png)
![](https://i.imgur.com/EwZF02M.png)
Figure 11: Mikrotik configuration
</center>


:::info
3 Test the Fault Tolerance by stoping one of the cables and see if you have any downtime.
:::

Let's test how does it work, by deleting some of the links in the process of pinging
In order to do that I ping from the ```WEb``` to the ```192.168.122.1``` cloud interface

<center>

![](https://i.imgur.com/jsgvXPr.png)
![](https://i.imgur.com/1JDbl74.png)
Figure 12: Pinging from the Web to the cloud

</center>

As you may see the first time some package losses happen, but the second time does not
I also captured traffic from the second attempt by using wire shark

<center>

![](https://i.imgur.com/3HSQnvx.png)
Figure 13: Traffic captured in the ping process
</center>

:::info
4 Disable STP on the Switches under Internal
:::

SPT translates as spanning tree protocol, and it is used in order to prevent loops in the network, it is done by simply choosing the root switch, that will tell the others what path they should be using
In figure 14, you may see a packet of the SPT

<center>

![](https://i.imgur.com/HPl48Dx.png)
Figure 14: STP packets in the link
</center>

In our case ```internal``` switch is selected as root, because it has the eldest MAc address

mac of the itdepatrment switch
0c:a5:c0:4b:00:00

mac of the management switch
0c:df:e5:f7:00:00

mac of the internal switch
0c:cf:84:92:00:01

disabled STP protocol can cause serious problems if there are any loops in the network

<center>

![](https://i.imgur.com/SNkdfwS.png)
Figure 15: Absecnse of the STP tarffic in network, SPT disabled
</center>

:::info
6 Change the topology to have two paths as shown below:
:::

I added a new link connection in order to show how doe STP disables will affect our network

<center>

![](https://i.imgur.com/6Oxz7zc.png)
Figure 16: New topology
</center>


:::info
7 Capture the traffic send a broadcast ping request to the PCs connected to the Internal Network.
:::

Let's inspect what kind of reaction will happen in the network

1 What can you notice ?

Right from the moment of the connection new link, the Wireshark capture window exploded with the flood of packets. Also, the CPU of the work station is it became loaded on 100%
In figure 17 you can process the capturing while i am trying to ping from ITmanager to HR, and there is no such packet

<center>

![](https://i.imgur.com/F3wEwIf.png)
![](https://i.imgur.com/96UJvsp.png)
Figure 17: Effect of the STP turned off on wire shark and the CPU
</center>


2 Why did this happen ?

This happens because of the broadcast packets. They are sent to every port of the switch, which means if the switch< that have received that packet, will also send to the all ports, in several seconds an avalanche of broadcast packets will fill the network

3 What are the implications of this on the network ?

What is happened is a broadcast storm. Because of the amount of packets, the CPU of the switches are loaded at 100%, and the network itself cant work properly

:::info
6 Enable back STP on the Switches and do the experiment again.
:::

Let's enable STP back and try to repeat the ping process from the ITmanager to the HR, you can see the process of ping in  picture 18

<center>

![](https://i.imgur.com/Xc2HxNd.png)
Figure 18: Pinging from the ITmanager to the HR
</center>

1 Can you see STP trafic ? Explain it breifly.

In the picture, you can see that the ping command is working again, and there are STP packets again

<center>

![](https://i.imgur.com/FEEIYeD.png)
Figure 19: Wire shark capturing
</center>

2 Configure the switches to have the Internal as the Root switch.

Our internal switch is already a root switch, but in case if he were isn't, there is a command that allows us to show and configure STP settings

In order to change the configuration of STP on particular bridge, i preformed
```
net show bridge spanning-tree
```

<center>

![](https://i.imgur.com/VZZLpIB.png)
Figure 20: spanning-tree configuration
</center>

In order to change configuration of STP on particular bridge i preformed

```net add bridge stp treeprio 8192```

<center>

![](https://i.imgur.com/69NkuDn.png)
Figure 21: spanning-tree configuration changing
</center>


:::info
7 Would we need STP between routers ?
:::

STP is a 2 layer protocol, so used mainly by switches, so the routers don't need it, but for example, used in this lab router MikroTik  have a protocol RSPT on the bridge on by default

<center>

![](https://i.imgur.com/CXd1jo5.png)
Figure 22: RSPT on hte mikrotik
</center>

---

1: [Основы компьютерных сетей. Тема №6.](https://habr.com/ru/post/319080/)
2: [Cumulus Linux 3.7](https://img-en.fs.com/file/user_manual/cumulus-linux-user-guide.pdf)