###### tags: `AN`
:::success
# AN Lab 1 Assignment: QoS
**Name: Radomskiy Ilia**
:::




## Task 1: Prepare your network topology
:::info
Prepare a simple network consisting of at least one router and two hosts. About four hosts in the network are most optimal. You also might need Internet access for the hosts:
:::

I will configure the topology that is shown in the picture below

<center>

![](https://i.imgur.com/VDIHYsQ.png)
</center>

I will not show the full configuration process, just final results, like:
Ip addresses on the router
![](https://i.imgur.com/FPL2NSd.png)

How I configured hosts and their availability to the internet
![](https://i.imgur.com/16LDjDF.png)


---
## Task 2: QoS learning & configuring

:::info
1. Сlass of Service (CoS), Type Of Service (ToS), Differentiated Services Code Point (DSCP), Serialization, Packet Marking, Tail Drop, Head Drop, The Leaky bucket algorithm, The Token Bucket Algorithm, Traffic shaping, Traffic policing?
:::

**Сlass of Service** (CoS) - ability to prioritize  traffick depending on its class 

**Type Of Service** (ToS) - byte with the priority of the packet

![](https://i.imgur.com/V5LYtDM.png)
ToS header in IPv4

**Differentiated Services Code Point (DSCP)** - space that contains code, this code describes class of the service and priority of deleting

**Serialization** - time within that device can unpack (packet to bits) and put a link to the next node

**Packet Marking** - belonging packet to a specific class, this is written in the header

**Tail Drop** - end of the queue is deleted when the buffer is overloaded

**Head Drop** - packet at the beginning is deleted when a new packet comes to the overloaded buffer

**The Leaky bucket algorithm** - bucket, in this case, is buffer, rectangular - packets, hole in the bucket is equal to the throughput of the channel

**Traffic shaping** - limiting the traffic speed by using buffering of excess traffic. (packets go to the buffer than shaper take packets of it with the particular frequency)

**Traffic policing** - limiting of the traffick speed with deleting of excess packets


<center>

![](https://i.imgur.com/8mrDxpb.png)
The Leaky bucket algorithm
</center>

**The Token Bucket Algorithm** - in this case, packets `have to pay` with tokens for the the ability to go through some node, amount of the tokens depends on the buffer size(bucket), if there to place for new packets (another buffer is overloaded), packets are dropped

[QoS - Habr](https://habr.com/ru/post/420525/)

:::info
2. speed limitation (traffic shaping) between the two hosts.
:::

In this task, I am using iperf3
The command for the host to become a server, go to the listening mode
`iperf3 -s`

Command for the host to connect to the server, in order to measure the speed
`iperf3 -c (adress) 192.168.4.2 -p 5201 -i(interval) 5 -t 30`

This commands is used in order to create rules on speed limitation
`queue simple add name=1 target=192.168.1.2 max-limit=1M/1M`


`queue simple add name=2 target=192.168.2.2 max-limit=500K/500K`
You can see how does this rule affect speed limits in the pictures below

:::info
3. Run a bandwidth testing tool, see what is the max speed you can get and verify your speed limitation. Compare the speed between the different hosts.
:::

In this case, maximum available speed is shown

![](https://i.imgur.com/UTomXtB.png)

In this case speed limit is 1M/1M for host 1

![](https://i.imgur.com/47OKRSY.png)

In this case speed limit is 500K/500K for host 2

![](https://i.imgur.com/kn5heFi.png)

:::info
4. While your bandwidth test is still running, try to download a file from one host to the other host and see what is the max speed you can get. If you have more than two hosts on the network, play around with different speed values and show it.
:::

Firstly I will try to download wile from the internet  with the command
```
wget https://mirrors.layeronline.com/linuxmint/stable/20.2/linuxmint-20.2-cinnamon-64bit.iso &
```
And measure speed with
```
sudo iftop
```
this line is user to rewoke background process
```
fg
```

then I was trying to do the same, but with downloading from host 3

this line is used for the creation of fie size 50 Mbytes
```
fallocate -l 50M gentoo_root.txt
```

downloading from host 3 via ssh
```
scp ubuntu@192.168.3.2:/home/ubuntu/gentoo_root.txt /home/ubuntu/gentoo_root.txt
```

in this case, I am downloading Linux image on the 3 hosts, host 1 have the rule with a speed limit of 1Mbytes/s, host 2 have the rule with a speed limit of 500kbytes/s, host 2 have no rules

<center>

![](https://i.imgur.com/yWP69AO.png)
Host1
</center>
<center>

![](https://i.imgur.com/ZVs2S5G.png)
Host2
</center>
<center>

![](https://i.imgur.com/2DWiX7m.png)
Host3
</center>

In the two pictures below you can see speed measurement with and without downloading

<center>

![](https://i.imgur.com/8W7IJ21.png)
with downloading
</center>
<center>

![](https://i.imgur.com/kn5heFi.png)
without downloading
</center>

as you can see there is a small effect on speed tests because of downloading

in this case below I am downloading the file from host 3, 

<center>

![](https://i.imgur.com/SxUvt6v.png)
host 1 is downloading, speed limitation is 1Mbytes/s
</center>

<center>

![](https://i.imgur.com/FPo9wF9.png)
host 2 is downloading, speed limitation is 500Kbytes/s
</center>

:::info
5. Deploy and verify your QoS rules to prioritize the downloading of a file (or any other scenario) over the bandwidth test
:::

In this task, I am trying to implement prioritization rules

In the picture below you can see how 2 hosts are downloading files, and there is a difference in downloading speed

<center>

![](https://i.imgur.com/PG4Ggwf.png)
downloading speed before rules implementing
</center>

After the first measurements I have added prioritization rules on the router, downloading for the host 2 (ether4), is prioritized, so the user should not suffer this speed decreasing

```
queue simple add target=ether4 dst=ether3 priority=1/1
queue simple add target=ether4 dst=ether2 priority=8/8
queue simple add name=3 target=192.168.3.2 max-limit=1M/1M priority=1/1 total-max-limit=1M
```

<center>

![](https://i.imgur.com/jK1d1xL.png)
Implemented rules shown on mikrotik
</center>

In the picture below you can see new speed measurement

<center>

![](https://i.imgur.com/NbhyPV9.png)
downloading speed after rules implementing
</center>

by comparing tests, we can see that speed for host 2 (43900 port) is not decreased, but on the other host is decreased

The same rules can be implemented in a different ways, I case below I am using `queue simple`, but if you want to set more particular rules, then you need to use a combination of `firewall rules` and `queues`

In the three pictures below you can see what kind of rules was implemented
in `firewall > mangle` marking any traffic that comes from any network to the host 3
![](https://i.imgur.com/P2xaM4c.png)

Then we are creating two queues for incoming and outcoming traffick
![](https://i.imgur.com/908rcl0.png)

Lastly we are adding speed limitations on the marked packets
![](https://i.imgur.com/zCOinW9.png)

So, in the picture below you can see, how does it effect on the network, every participant downloading is shearing bandwidth
![](https://i.imgur.com/ET00gqS.png)


:::info
6. What is the difference between the QoS rules to traffic allocation and priority-based QoS? Try to set up each of them and show them. In which tasks of this lab do you use one or the other?
:::

Allocation is when we are working with the bandwidth in general, so rules are applied to every traffic according to rules.
Prioritization is when one type of traffic (chosen by the administrator) is primary, or have an order of prioritization

:::info
7. Choice and install any tool that you like for bandwidth control/netflow analysis/network control & monitoring. Play around with the tool features and show the different network metrics via GUI.
:::

I will be using `ntopng` for network monitoring, below you can see pictures with different tests, and possible usage cases

In the first picture, we can see network usage by one host, on downloading and uploading. Significant increase in happening because I started downloading file

<center>

![](https://i.imgur.com/S9y4cEv.png)
</center>

On the second picture, we can see different pie charts, on sent, received and protocol usage by host

<center>

![](https://i.imgur.com/b1VEcfC.png)
</center>

on this tab, we can see other information about the host
![](https://i.imgur.com/eZ4I4xZ.png)

we also can see what kind of web sites was visited by our user
![](https://i.imgur.com/ZhT8Pv4.png)


:::info
**Bonus**: try to apply and verify any QoS rule/mechanism that you are interested in (this may be your new idea or one of the QoS configuration that you set up in the lab earlier).
:::

In QoS we can implement such thing as scheduling, working on particular time
I have added a rule that decreases the speed of downloading to 500kbyte/s but only between 00:00:00 and 19:20:00 after 19:20:00 speed is equal to the 1M (a limit that was set on the previous task)
In the pictures below you can see rules on the Mikrotik terminal
<center>

![](https://i.imgur.com/Ibo3fWq.png)
</center>

and two speed test

<center>

![](https://i.imgur.com/YwMDSmC.png)
when schedule  rule is working

![](https://i.imgur.com/6ZCLF7F.png)
when schedule  rule is not working
</center>


:::info
8. packet drops can occur even in an unloaded network where there is no queue overflow. In what cases and why does this happen?
:::

Yes:
Network or one of the core devices is down
RJ-45 cable is located near to the high voltage cable or near to "noisy" cable, and/or there is no protection on the cable
Misconfigurations or configuration is done wrong


---
## Task 2: QoS learning & configuring

:::info
1. How can you check if your QoS rules are applied correctly? List and describe the various methods.
:::

1. Rules are doing what they are meant to be
2. Packets are following the rules that you have made
3. In the IP header there is a ToS mark
4. `print stats` (Mikrotik) or something familiar on the vendor of another type

<center>

![](https://i.imgur.com/5ghGHWx.png)
print stats command in mikrotik terminal
</center>

:::info
2. Try to use Wireshark to see the QoS packets. How does this depend on the number of routers in the network topology?
:::

We can locate the QoS packet in Wireshark, you can see it in the picture below

<center>

![](https://i.imgur.com/DWqaJSb.png)

</center>

---
## **Bonus**: Two routers 

:::info
1. Edit your network scheme to use 2 hosts and 2 routers, each one of them has a different subnet, and the 2 hosts should be able to reach each other (direct connection between the routers and static routes is more than enough) and redo task 2.
:::

On the picture below you can see the topology that I have built for this task, I have implemented ipv6 on this topology

<center>

![](https://i.imgur.com/DNB2LnR.png)
</center>

Router 2 was configured like it shown on the picture below

<center>

![](https://i.imgur.com/ogkY8ld.png)
routes
![](https://i.imgur.com/L7IT0ta.png)
adsresses of interfaces
</center>

Host 8 was configured in the way it is shown below, other hoast were configured in the same way

<center>

![](https://i.imgur.com/M7lQ2tS.png)
8 host's neplan
</center>

The next 6 pictures are representing tests that has been made in different situations
This pair of pictures was made to measure maximum speed, so there is no rules implemented

<center>

![](https://i.imgur.com/G86W1UX.png)
client side
![](https://i.imgur.com/i4cmPGG.png)
server side
</center>

In this case, I set a rule that decreases speed to the 1Mbyte/s, on the router 2

<center>

![](https://i.imgur.com/k83uLpL.png)
rule that decrese speed for the host 5
![](https://i.imgur.com/2Wdky1a.png)
speed test, server side
</center>

In this case, I set a rule that decreases speed to the 1Mbyte/s, on router 3, the rule that was turned on previous is disabled

<center>

![](https://i.imgur.com/t7yjG8j.png)
rule that decrese speed for the host 8
![](https://i.imgur.com/QqKok8D.png)
Speed test, server side
</center>

The 4 Pictures above show that, if we are creating a rule on the interface, then you need to implement only 1 rule (on 1 router), and it will be working

Now I will be measuring connection speed, and after some time I will start downloading

I will be using this command in order to download while from host 5

```
scp -6 ubuntu@\[fc02:ca09:2bc5:a2f5::5\]:/home/ubuntu/gentoo_root.txt /home/ubuntu/
```
this command is used to test the speed

```
iperf3 -c (server)
iperf3 fc02:ca09:2bc5:a2f8::8 -p 5201 -i 5 -t 30 (client)
```

<center>

![](https://i.imgur.com/m7q2yBn.png)
start of downloading during speed test
</center>

In the pictures below I am setting up rules that gives priority to download from host 5 to host 6, 

In the picture below you can see downloading speed test before rules were implemented
![](https://i.imgur.com/R74T4dO.png)

In this picture, there are rules that were implemented
![](https://i.imgur.com/aA9jTsk.png)


In the picture below you can see downloading speed test after rules were implemented
![](https://i.imgur.com/lG5iBCE.png)



:::info
2. Do you need to write QoS rules for each router? Is there a way to let the other routers know that this packet has more priority over the other packets?
:::

If we are setting up a rule that decreases speed on the interface, or IP address, it cant go above this on any other router, like in the example above
I couldn't succeed to locate dcsp packets (with prioritization) after the router, which might mean that rules are implemented only on the router

:::info
3. Try to deploy your network scheme over IPv6
:::

It was implemented in subtask 1


---
## References:


