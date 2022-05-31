###### tags: `INR`
:::success
# INR Lab 5 Assignment: BGP
**Name: Radomskiy Ilia**
:::

**hints**
hard and soft reset

## Task 1: Preparation:

:::info
a. Select a virtual routing solution that you would like to try. For example (Mikrotik, vyos, Pfsense).
:::

Similar to the previous labs, the virtual routing solution of my choice will be Mikrotik

:::info
b. GNS3 already has a template for these routers (Mikrotik, vyos, Pfsense), try to use these templates as it will save you a lot of time and troubleshooting.
:::

Some of the templates can be found in the ```browser all devices > new device > "chose your device"```
![](https://i.imgur.com/OyBboIF.png)

:::info
c. Try to draw a network scheme before you start the lab. This will help you in the deployment phase.
:::

I will use the topology from the previous, lab, you can see it in the picture below

:::info
d. The network scheme should include at least two networks, each one of them should have at least 3 routers, these routers can be the same routers from the OSPF lab.
:::

<center>

![](https://i.imgur.com/oQi7D95.png)
Topology of the lab
</center>

:::info
e. This lab should be done in teams of 2, 3, or 4. As long as each team member uses a different
network device
:::

Some of the topologies of my groupmates

<center>

![](https://i.imgur.com/g8PqOsX.jpg)
Vladimir
</center>

<center>

![](https://i.imgur.com/tAZTMXM.png)
Alexander
</center>

<center>

![](https://i.imgur.com/CBmSovz.jpg)
Alisher
</center>

<center>

![](https://i.imgur.com/cdRfgNP.jpg)
General view
</center>



:::info
f. Connect one of your OSPF ASBR router interfaces (BGP interface) to your physical interface (bridge can also be used)
:::

In order to have communication between the GNS3 network and the topology of the other students, I used just my physical interface ```eno1``` via the cloud in GNS3

:::info
g. Agree with other teams on a subnet that your team will use (DO NOT USE 10.1.1.0/24)
:::

We have an agreement on using ```217.1.1.0/24``` network with my groupmate Vladimir
other students use networks that are represented on the picture with general view of the stand

:::info
h. Check that you can ping your teammates ASBR router from your ASBR router
:::

WHen i am connected to the cloud i am able to reach my groupmate Vladimir
<center>

![](https://i.imgur.com/Gym64w4.png)
Pinging  to the ASBR router of my partner
</center>



---
## Task 2: Deployment

:::info
a. Define an AS number that your ASBR router will use, again agree with the other teams on which. AS number each team will use (64512 to 65534)
:::

AS (autonomous system) system to networks and subnets that have one management policy may be one organization or region
With the command listed below i am adding new record instance to bgp
```routing bgp instance add router-id=10.255.255.100 as=65444 name=bgp1```

:::info
b. Enable BGP and start advertising your OSPF network to your peer.
:::

In order for others to see my devices I am adding new several records in the BGP instance:
```client-to-client-reflection=yes```
Thir record required if we need reflection between clients in one cluster
If I want to share my network with others I will type this command
```redistribute-osfp=yes```
All records for the instance look like this
```routing BGP instance add router-id=10.255.255.100 as=65444 name=bgp1 client-to-client-reflection=yes```

<center>

![](https://i.imgur.com/ZuNCqbf.png)
BGP settings
</center>

:::info
c. Can your peers reach your internal subnets? And can you reach their internal subnets?
:::

So, when BGP in on, we are able to reach to networks of my groupmates


<center>

![](https://i.imgur.com/pmSQP1e.png)
Vladimir
</center>

<center>

![](https://i.imgur.com/P0QMSSQ.png)
Ivan
</center>

<center>

![](https://i.imgur.com/lShzubu.png)
Alisher
</center>

:::info
d. How can the OSPF Internal router know about your peer’s OSPF Internal router? One way is to redistribute BGP routes into the OSPF routing table, but is this a practical method? why?
:::

No, this is not convenient because there are a lot of routes on the internet, and store the BGP table (that is huge) in the OSPF table is a waste of memory. But you still can allow knowing about your internal OSPF with the virtual tunnelling (BGP MPLS VPN)

---
## Task 3: Verification

:::info
a. How can you check if you have an established status with your peer?
:::

Amount of peers are available in the ```routing bgp peer```, on the picture below you can see every peer that was added by me. In this table there is only one mark ```E```which means ```established```, that. In our big topology, i have only one connection - with the Vladimir

<center>

![](https://i.imgur.com/q2OXjCP.png)

![](https://i.imgur.com/Ix44mep.png)
Peers of my ASBR
</center>

:::info
b. How can you check in the routing table of ASBR and OSPF Internal routers to see which
networks did you receive from your neighbors?
:::

Every route, that was received from my neighbour, is listed in the routing table, which can be found in the ```ip route print```, you can see it in the picture below. This table is huge, and have almost 70 records, this is because Vladimir is sharing every existing route that he has on his table

<center>

![](https://i.imgur.com/fl60kU5.png)
![](https://i.imgur.com/5lx6j1a.png)
</center>


:::info
c. Use traceroute to verify that you have full BGP and OSPF connectivity.
:::

<center>

![](https://i.imgur.com/YljPOnl.png)
Traceroute to the others network
</center>

---
## Task 4: Transit

:::info
a. Peer with other teams and send them the routers that you receive from your peer in task 2
:::

In order to create a connection with the new peer from the other network, I added a new BGP instance, it also needs to be configured new network and the IP address on the interface, in order to have a connection

![](https://i.imgur.com/voOg4du.png)

At this point, ASBR have a connection with two routers, but they are cant ping each other, in order to improve that ```masquarad``` setting need o to be added, on the Mikrotik



:::info
b. Can the new peer reach your network and your teammate network through you?
:::

![](https://i.imgur.com/5gK7OmK.jpg)


---
## Task 5: ISP (Bonus)

:::info
a. Become a transit for all/most of the networks that are available.
:::



---
## References:
