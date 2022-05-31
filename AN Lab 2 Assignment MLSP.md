###### tags: `AN`
:::success
# AN Lab 2 Assignment: MLSP
**Name: Radomskiy Ilia**
:::


---

## Task 1: Prepare your network topology

:::info
1. In the GNS3 project, select and install a virtual routing solution that you would like to use: for example, Mikrotik, Pfsense, vyos.
:::

I will be using MikroTik  for this lab

:::info
2. Prepare a simple network consisting of at least three routers and two hosts (for example, a bus topology with dynamic routing). Each one of them has a different subnet. Your network has to have routing protocols configured. That's why you can use your OSPF lab project from INR course.
:::

My network will be simple and consisting only of 3 routers and two hosts, you can see it in the picture below

<center>
    
![](https://i.imgur.com/zVF0Vhk.png)

</center>

On the pictures below you can see tables, that have been configured for the working OSPF. In the last picture there test between `PE1` and `PE2`, and a routing table that has been configured because of OSPF

<center>
    
![](https://i.imgur.com/LX6d67e.png)
![](https://i.imgur.com/qYZdN1X.png)
![](https://i.imgur.com/6lMVe93.png)

</center>


<center>
    
![](https://i.imgur.com/au1peZB.png)

</center>
    
[](https://www.timigate.com/2019/03/using-mikrotik-mpls-with-vpls-to-connect-two-offices.html)
[](https://www.manitonetworks.com/mikrotik/2016/5/24/mikrotik-mpls-with-vpls)

---
## Task 2: MPLS learning & configuring

:::info
1. Briefly answer the questions or give one-line description what is it: LSP, VPLS, PHP, LDP, MPLS L2VPN, CE-router, PE-router, LSR-router?
:::

**LSP** — Label Switched Path - the path between ingress and egress, the path of a packet

**VPLS** - Virtual Private LAN Service - way to connect, point-to-point, different parts of the network over IP or MPLS, using labels. Users in this network share the same broadcast domain

**PHP** - Penultimate Hop Popping, the special label on the packet, maps labels always `ingress` and `poping` on routers during transmission, but PHP suggest `pop` MPLS label on the `Penultimate` router. For this end is used special label 3.

**LDP** - Label Distribution Protocol, protocol, that allows exchanging information with the other routers about routing

**MPLS** - a method that allows exchanging over the network with the help of labels

**L2VPN** - method of creation Virtual Private Networks over the second layer of OSI

**CE-router** - Customer Edge router, that connected to the provider

**PE-router** - Provider Edge router, exactly to this router CE is connected

**LSR-router** - Label Switch Router, any router in MPLS network, but not border router

[Basic MPLS](https://habr.com/ru/post/246425/)
[MPLS L3VPN](https://habr.com/ru/post/273679/)
[MPLS L2VPN](https://habr.com/ru/post/315028/)

:::info
2. Configure the MPLS domain on your OSPF network, first without authentication.
:::spoiler
Hint: it is assumed that OSPF has already been configured previously.
:::

On the pictures below you can see tables, that have been configured for the MPLS to work
For MPLS and LDP I have configured 2 interfaces for it to be used
Then I turned LDP on, and have added neighbours to it
I have done it for every router

<center>
    
![](https://i.imgur.com/DJxjX96.png)
    Router P
![](https://i.imgur.com/fgwIrCJ.png)
    Router PE1
![](https://i.imgur.com/gYqcw7m.png)
    Router PE2

</center>


:::info
3. Enable authentication (what kind of authentication did you use)? Make sure that you can ping and trace all your network.
:::

To enable authentication I can perform the command listed below, I will be using md5 method


```
routing ospf interface add interface=ether1 authentication=md5 authentication-key=st10 authentication-key-id=2
```
I have added authentification for PE1 and P routers


<center>
    
![](https://i.imgur.com/0U6TKQa.png)

</center>

You can see packets with encrypted OSPF hello messages headers in the picture below

<center>
    
![](https://i.imgur.com/1fEwBVQ.png)

</center>
    
---
## Task 3: Verification

:::info
1. Show your LDP neighbours.

:::spoiler
Hint: in the case if you have some problems... think about whether there are enough routers and subnets for MPLS to pass the route?
:::

LDP neighbours can be shown with the command `mpls ldp neighbor print`, you can see results in the pictures below for every router

![](https://i.imgur.com/0V2d1SP.png)
![](https://i.imgur.com/ENmryMk.png)
![](https://i.imgur.com/FtA34nK.png)


:::info
2. Show your local LDP bindings and remote LDP peer labels.
:::

In this table we can see local-bindings of our configured network, that information can tell us that we are marking some of the packets with the specific label,  in range `5000-9999` for `P` router

<center>
    
![](https://i.imgur.com/7T2TFJl.png)
![](https://i.imgur.com/tI6lfe4.png)
![](https://i.imgur.com/fxxTfDw.png)

</center>

We also can see what labels are set by another router with, `mpls forwarding-tabel`. In our case, we can see labels only of `P` router in the `PE1` and `PE2` routers, 5001 and 5000 correspondently

<center>
    
![](https://i.imgur.com/m9KYAyV.png)
![](https://i.imgur.com/mofdRD1.png)
![](https://i.imgur.com/pUbDMko.png)

</center>

:::info
3. Show your MPLS labels.
:::

MPLS labels can see seen in Wireshark, for example. You can see it in the picture below, there are two additional headers: `Multiprotocol Lable Switching header`


![](https://i.imgur.com/9iQBnbD.png)

:::info
4. Show your forwarding table.
:::

The forwarding table can be shown by typing `mpls forwarding-table`
In the picture below you can see routes for 3 routers


![](https://i.imgur.com/6m8JgzC.png)
![](https://i.imgur.com/IRObr1r.png)
![](https://i.imgur.com/jY47WPi.png)


:::info
5. Show your network path from one customer edge to the other customer edge.
:::spoiler
Hint: you can use traceroute command.
:::

`PE1` and `PE2` are customer routers, we can `traceroute` between them if we want to show them being able to communicate, both routers, at the first stage go to the `P` router, but with using different networks

<center>
    
![](https://i.imgur.com/OyLWCPe.png)
![](https://i.imgur.com/2If90Pk.png)

</center>

---
## Task 4: MPLS packets analysis

:::info
1. Can you use Wireshark to see the MPLS packets?
:::

yes, we can, the result is shown in the pictures below

:::info
2. Look deeper into the MPLS packets: can you identify MAC address, ICMP, Ethernet header or something else useful?
:::

Not exactly, in MPLS headers we can see MPLS labels as most important information - two pictures below

![](https://i.imgur.com/eyGDzFf.png)
![](https://i.imgur.com/NuJMjHJ.png)

But in the case with LDP we can see LSR's id, IP of loopback interface, it is something more

![](https://i.imgur.com/Zjh9ppm.png)

Another useful information like MAC, IP we are getting from different headers, like `Ethernet II` or `Internet Protocol`, or another packet

---
## Task 5: VPLS

:::info
1. Configure VPLS between the 2 hosts edges.
:::spoiler
Hint: you should connect the hosts connected to the router in one subnet to the router from the second subnet without changing the topology physically. To do this try to configure VPLS that runs on L2.
:::

VPLS is configured in the way it is shown in the 2 pictures below. For setting it up I have added two things - VPLS interface, and add interfaces in the particularly created interface.

<center>
    
![](https://i.imgur.com/IAt2obl.png)
![](https://i.imgur.com/6OVf7od.png)

</center>
    
:::info
2. Show your LDP neighbors again, what has been changed?
:::

after I have configured VPLS, the MPLS neighbour table has changed, you can see them in the picture below.

<center>
    
![](https://i.imgur.com/H9RrCfp.png)
![](https://i.imgur.com/pI06UfR.png)
new neighbor tables
</center>

You can compare it with the tables that were before the VPLS configuration

<center>
    
![](https://i.imgur.com/0V2d1SP.png)
![](https://i.imgur.com/ENmryMk.png)
![](https://i.imgur.com/FtA34nK.png)
old neighbor tables
</center>
    
:::info
3. Find a way to prove that the two customers can communicate at OSI layer 2.
:::

First, I have created a new topology, which is shown in the picture below. This is a simple network, hosts are located in different networks: ubuntu1 - 192.168.1.10, ubuntu3 - 192.168.2.10.

<center>
    
![](https://i.imgur.com/5uSKJE3.png)

</center>

I assume that, if communication is happening on the second layer, it is possible to make `apring` (ping with apr protocol) from one host to another, in case it is third layer communication, it is not possible

<center>
    
![](https://i.imgur.com/AvEzeyG.png)
1 pinging - succes
2 arping - fail
</center>

You can see the Wireshark interception in the two pictures below. `Arp` packets in pictures it is communication between host and router

<center>

![](https://i.imgur.com/obGrB8x.png)
![](https://i.imgur.com/a84G9XR.png)
1 pinging - succes
2 arping - fail
</center>

Second, I use my VPLS set up the network, you can see it in the picture below, and perform the same operations

<center>
    
![](https://i.imgur.com/RRWI7iG.png)
1 pinging - succes
2 arping - succes
</center>

In this case, ping and `arping` are successful

<center>
    
![](https://i.imgur.com/NWXQZkL.png)

</center>

<center>
    
![](https://i.imgur.com/snQyjII.png)
![](https://i.imgur.com/gbbu13R.png)
1 pinging - succes
2 arping - succes
</center>

Another way is to `traceroute` from one host to another, you can see it in the picture below, in this case, we cant see the router's IP

<center>

![](https://i.imgur.com/iZNEXl2.png)

</center>

:::info
4. Is it required to disable PHP? Explain your answer.
:::

PHP - Penultimate Hop Popping is the nutshell is working like that:
1 packet has arrived on the router

2 Router looks up in headers and checks sender MAC address
2.1 router might already know MAC (so step 3)
2.2 router might not know it (so write table)

3 Router looks up in headers and checks receiver MAC address
3.1 router might already know MAC (so to look on which interface is located destination to send it)
3.2 router might not know it (broadcast with the labels for each PE router)

4 after packet arrives at destination router, all labels are removed, and router acts as a switch

So in this correspondence, i think,  it doesn't matter for PHP is enabled or not

---
## **Bonus** - VPWS

:::spoiler
There are two approaches to build L2VPN: Point-to-Point (VPWS) or Point-to-Multipoint (VPLS). You
studied VPLS a little earlier. Now you may learn the concept of PW (Pseudowire) and VPWS (Virtual
Private Wire Service technology). VPWS is an L2VPN technology that transmits layer 2 services,
simulating the main characteristics and functions of services such as ATM, Ethernet and some others.
:::

:::info
1. Rebuild your topology to get a pseudowire P2P that emulates OSI layer 2.
:::
To this end, I have established an l2tp connection between 2 routers

:::info
2. Repeat the steps from Task 5, but now with VPWS.
:::

First - a pool of available IP addresses
```
ip pool add name=LAN-Ip-Pool ranges=192.168.10.100-192.168.10.150
```
second PPP profile that is configured in the picture below

![](https://i.imgur.com/pkyJHIX.png)

Next, enabling l2tp interfase to `on` stattus

![](https://i.imgur.com/8fMsJ0D.png)

Last one, to establish a server-client connection  between two routers

![](https://i.imgur.com/Var66Gi.png)
![](https://i.imgur.com/U5zA1FR.png)

Now we are able to test our network

![](https://i.imgur.com/bpVB66M.png)

We also can see how does it affect on the Wireshark

![](https://i.imgur.com/QWNNme1.png)

---
## References:



