###### tags: `INR`
:::success
# INR Lab 4 Assignment: OSPF
**Name: Radomskiy Ilia**
:::




## Task 1: Prepare your network topology
:::info
In the GNS3 project, select and install a virtual routing solution that you would like to use: Mikrotik (recommended), Pfsense, vyos and so on.
:::

In this work, I will be using MikroTik routers, as well as build-in VPCS and switches

:::info
Prepare a simple network consisting default route of at least 3 routers, each one of them has a different subnet, and they should be able to reach each other (for example by a
switch/router in the middle of a bus topology). A couple of variants are below:
:::

In order to test OSPF, I build the network listed in the picture

<center>

![](https://i.imgur.com/EaWiidQ.png)
Topology of the network
</center>

With the basic (IP on the interfaces) configuration they are able to ping each other.
you can see on the picture below two routers pinging every router in the network, at this point routers are able to ping VPCS

Basic routers configuration
```
ip address add address=192.168.1.2 interface=3
ip address add address=10.10.1.1 interface=1
ip address add address=10.10.4.2 interface=2
```

<center>

![](https://i.imgur.com/IkwMa3b.png)
![](https://i.imgur.com/0jq22Jz.png)
Ping between routers
</center>

---
## Task 2: OSPF Learning & Configuring

:::info
1. Deploy OSPF in your chosen network topology.
:::

OSPF stands for Open Shortest Path First, it is a protocol of the dynamic configuration, it is based on link state technology

In order to set up OSPF i have added routing instances that surrounds my routers, in case of the Router 1, those networks are ```192.168.1.0``` and ```10.10.1.0``` and ```10.10.4.0```

and added them as a network of the OSPF with

```
routing ospf network add area=backbone network=*(networks that surround  router)
```

For the Mikrotik 1 those commands are
```
routing ospf network add area=backbone network=10.10.1.0/24
routing ospf network add area=backbone network=10.10.4.0/24
routing ospf network add area=backbone network=192.168.1.0/24
```

From this point OSPF is working, some of the instances are set on the standard values, and can be modified later on
On the picture, we might see routing tables of all 4 routers, as well as IP addresses

<center>

![](https://i.imgur.com/thrqrd3.png)
Routing tables and ip tables
</center>

So pinging is available to the VPCS's
On the pictures are shown the progress of pinging from PC3 and PC1< to every other VPSC on the topology

<center>

![](https://i.imgur.com/b6L4qK4.png)
Ping from PC3

![](https://i.imgur.com/gM2HaPs.png)
Ping from PC1
</center>

:::info
2. Which interface you will select as the ```OSPF router ID``` and why?
:::

The router ID is used in order to set up the main router called ```designated router```
In order to do this, i created a loopback interface and giving IP address to him, by typing several commands

```
interfaces bridge add name=loopback
chosen router id
ip addresses add addresses=10.255.255.1/32 interface=loopback
```
From this point, we might choose this interface as router-id, before configuration is was set automatically, but sometimes, we have to change it

```
routing ospf instance edit number=0
10.255.255.1
```

in our case interface, 10.10.1.1 was set up as a default router
The highest priority is the network/address / 32 mask. The narrower the mask, the more priority;
The Default Route has the lowest priority 0.0.0.0/0.

<center>

![](https://i.imgur.com/Y67c9gY.png)
Wireshark capture
</center>

:::info
3. What is the difference between advertising all the networks VS manual advertising (perinterface or per subnet)? Which one is better?
:::

Advertise means to create a summary LSA (table) and advertise it to adjacent areas, or so to tell other routers about possible paths
Advertising all the networks means that your router will be Advertising every part in his routing table
Manual advertising is the opposite. This option would better be performed if you don't want to share information about routes

:::info
4. If you have a static route in a router, how can you let your OSPF neighbours know about it? Approve and show it in practice.
:::

To set up static router i added a new router in the topology

<center>

![](https://i.imgur.com/tpO0YMG.png)
New topology
</center>

I added some of the paths in the router, one to the network ```192.168.5.0``` another to the ```10.10.5.0``` via commands that are listed below, also i have added static route to the router 1

```
ip address add interface=ether2 address=192.168.5.1/24
ip address add interface=ether1 address=10.10.5.2/24
ip route add dst-address=192.168.0.0/16 gateway=10.10.5.1 
```

it is also need to be configured static router in the router 1 in order to VPCS being able to reach network ```192.168.5.0```

```
ip address add interface=ether4 address=10.10.5.1/24
ip route add dst-address=192.168.5.0/24 gateway=10.10.5.2
```

From this point it is possible to ping from one zone to another

<center>

![](https://i.imgur.com/vENAm9J.png)
</center>

But in case if we want to advertise this route to another router we make additional configurations. On router 1 I added

```
routing ospf instance edit number=0 redistribure_static_route
as_type_1
```

So, after OSPF shared this information we are able to see this route on the other routers

<center>

![](https://i.imgur.com/N8TXc6c.png)
![](https://i.imgur.com/Wd3SbGw.png)
routing tables on the router 3 and 4
</center>


:::info
5. Enable OSPF with authentication between the neighbours and verify it.
:::

authentication is used if we want to exchange routing tables in a secure way

it can be enabled by performing commands listed below

```
routing ospf interface add interface=ether1 authentication=md5 authentication-key=st10 authentication-key-id=2
```

```
routing ospf interface add interface=ether1 authentication=md5 authentication-key=st10 authentication-key-id=2
```

On the picture below you can see authentication configured between routers 1 and 2

<center>

![](https://i.imgur.com/JDXKEtP.png)
tables of the ospf interfaces
</center>

So right now every hello message is encrypted and we can see, how does it changed our packets

<center>

![](https://i.imgur.com/KL5my8r.png)
Packet without authentication

![](https://i.imgur.com/gHNy4vV.png)
Packet with authentication
</center>


:::info
6. ***Bonus:*** if one of the routers has multiple subnets, try to use route summarization.
:::

Summarization  is used if we want to set network in less work effort way
```
10.10.0.0/16
```
instead of

```
10.10.1.0/24
10.10.2.0/24
...
10.10.100.0/24
```

in order to do that, I added one line in the Mikrotik

```
/ip route
add distance=254 dst-address=192.168.0.0/16 type=\blackhole
```
<center>

![](https://i.imgur.com/HUvymUa.png)
New routing table with the summarized record
</center>


---
## Task 3: OSPF Verification

:::info
1. How can you check if you have a full adjacency with your router neighbor?
:::

adjacency means the amount of the routing tables synchronization
adjacency in Mikrotik can be checked with ```routing OSPF neighbour```

For example, in the picture below you may see how does progress of the getting adjacency i have done, a moment captured at the border of the OSPF exchange
There is a record that specifies amount of time ago that full adjacency happened

<center>

![](https://i.imgur.com/tL4nccd.png)
Progress of the syncranazation
</center>

:::info
2. How can you check in the routing table which networks did you receive from your neighbors?
:::

Near to the previous check routing, the table can be found, this table contains several important records, like the address of the network, the gateway that you need to send a packet in order to reach the network, the cost of the route, and interface of the router.
If the gate was marked  as 0.0.0.0 that means it is a local network
It is even better seen if you compare two tables/ before and after adjacency

<center>

![](https://i.imgur.com/neZjvVf.png)
routing table
</center>

:::info
3. Use ```traceroute``` to verify that you have a full OSPF network.
:::

In order to test the network with ```traceroute```I have added a new Ubuntu instance, and checked network in two cases, 1 when one line is broke (r1-r2), and when it the link is up
I will be using ```traceroute``` from ubuntu (network 1) to the VPCS 2 (network 2)

<center>

![](https://i.imgur.com/7Dkcj8A.png)
tracing route
</center>

at the picture, we can see all the gates that packege travelled through

:::info
4. Which router is selected as DR and which one is BDR ?
:::

Because each link between routers is in its own network, in each one of them it is one of two routers are set as DR another as BDR, you can see Wireshark capture in order to find out what interface of the router is DR in the 10.10.3.0 network (routers 3 and 4)

<center>

![](https://i.imgur.com/vJgmJvt.png)
wireshark capture
</center>

In my case:
DR 10.10.3.2
BDR 10.10.3.1

:::info
5. Check what is the cost for each network that has been received by OSPF in the routing table.
:::

The cost of the "roads" can be seen in the routing table that we have seen above

<center>

![](https://i.imgur.com/neZjvVf.png)
routing table with the cost of the paths
</center>




## ***Bonus***: Multi-Area network

:::info
1. In the case if until now every router was in area 0 , try to create more networks and assign them to different OSPF areas.
:::

in order to check how does OSPF deals with the different zone, I changed topology as it showed o the picture below

<center>

![](https://i.imgur.com/8yALULr.png)
New topology
</center>

So, after basic configuration, i added router 6 (network 192.168.6.0) in the area1 as shown on the picture below

<center>

![](https://i.imgur.com/1h7LXyy.png)
Setting on the router 6
</center>

Settings on router 2 stay the same except we add one new network 10.10.6.0

<center>

![](https://i.imgur.com/AFMfoaL.png)

</center>

It this point we can ping from VPCS 7
<center>

![](https://i.imgur.com/Ghz62pD.png)
Ping from VPCS 7
</center>

:::info
2. Why every area has to be connected to area 0 ?
:::

We need our zones to be connected with the backbone zone because routers have records connected with the backbone like you can see on the picture Setting on the router 6 above
we need to separate a network into zones because some routers are not strong enough to cover all zone, so in order to decrease impact of changes in the zone, the zone can be divided into several zones
also, we need to be connected to zone 0 in order to prevent loops


:::info
3. What can you do if you have an area X which is not connected to area 0, but to area Y ?
:::

The area that is not connected to the 0 zones is called stub-area. But there is an option in order to have a connection between such area and backbone, it is called ```virtual-link```
virtual-links used by transit zone (on both sides) in order to connect srub area and zone 0

:::info
4. Verify that area X can reach all the network.
:::

On this point, after configuration, VPCS 7 s able to ping every pc in the configuration

<center>

![](https://i.imgur.com/NaSA7g5.png)
ping from VPCS 7
</center>


