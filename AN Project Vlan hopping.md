###### tags: `AN`
:::success
# AN Project: Vlan hopping
**Name: Radomskiy Ilia**
**E-mail: r.ilya@innopolis.university**
**Course: Secure Systems and Network Engineering, Innopolis university**
:::

## Abstract:

**Abstract - In this project, I will create several laboratory stands to demonstrate how "Vlan hopping" works, in particular, two methods of this type of attack: "Double tagging" and "Switch spoofing". The project will result in a comparative analysis of the impact of this type of attack on different vendors of network equipment and methods of countering such attacks.**

## I. Introduction

VLAN’s are widely used in modern network architectures, they are really helpful when we need to connect two host devices to one network, even if they are geographically located elsewhere, or vice versa to place two hosts, that are physically connected to one switch, into different networks. This solution has some weaknesses, one of them is VLAN hopping.
VLAN hopping is an exploit that the original VLAN solution has. In general, the attack can be called VLAN hopping is the user gets access to the traffic of others VLANs. VLAN hopping Can be divided into two groups: “Switch spoofing” and “double tagging" bought off the attacks requires some conditions to the VLAN configuration.
“Switch spoofing” attack can happen when VLAN networks are configured to accept “trunk” tagged packets. 
“Double tagging” requires “native” VLAN tagging to be configured. If requests meet, then an attack can be performed.
In bought of cases, attacker pretend to be a switch device, configured in the trunk or native mode. And also in the bought cases, the attacker gets information about the VLANs, and about the packets that are transferred through the network.

## II. Research goals

* Research about VLAN hopping, possible methods of attacks, security measures against this type of attack
* Create several stands that can be representative examples of VLAN hopping attack
* MEasure How different vendors are facing this type of attack
* Is it possible to detect this type of attack, manually or automated?

## III. Related works

Investigating the Security of Nexus 1000V Virtual Switches in VMware ESXi Hypervisors [1]. This paper describes and answers one question "is the Cisco
Nexus 1000V virtualized with VMware’s
ESXi hypervisor vulnerable to the typical layer 2 attacks", the answer is yes, but not all of them.

VLAN hopping, ARP poisoning, and Man-In-The-Middle Attacks in Virtualized Environments [2]. The article describes how is it possible to perform 5 different types of attacks on the virtualized environment. A virtual network is based on switches of a different kind, and the base of the different kinds of virtualization platforms

## IV. Main Part

This part is divided into several parts:
In the `theory` section I will provide information about VLANs, how they work, attack methods on them, particularly about switch hopping and double tagging. Also, the method of topology construction will be described in the end of the `theory` section
After the first part, separate sections are suited for different types of vendors: Cisco, OpenvSwitch, and Cumulus.

### Task 1: Theory

:::info
1. VLAN
:::

VLAN - virtual local area network is an aggregation method [3]. With the implementation of which hosts can be connected to the same logical network, or located in different networks despite being connected to the same physical switch.
In table 1 you can see how does VLAN tagged internet frame look like. Tags are used by switches or routers to identify the destination of the packet. But the last switch removes all tags from the packet, and after that sends the packet to the destination user. If we want to communicate with the physically local users, then we can use an access port, it uses no tags. If we want to establish communication between two switches with the VLAN configured on them we need to use a trunk connection.

Table 1. Ethernet frame of 802.1Q

<center>

    Frame 802.1Q
    
| 8 bytes | 6 bytes | 6 bytes | 4 bytes | 2 bytes | 46-1500 bytes | 4 bytes |
| -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| Preamble | Dest. MAC | Source MAC | 802.1Q tag | ether type | data | CRC |

802.1Q tag in a closer look
| 2 bytes | 3 bits | 1 bit | 12 bits |
| -------- | -------- | -------- | -------- |
| TPID (Tag Protocol ID) | PCP (priority Cod Point) | CFI (canonical format indicator) | VID (VLAN ID) |

</center>

There are a lot of types of layer 2 attacks, some of them can be performed in order to exploit VLAN networks, but we will discuss only two of them `Switch spoofing` and `Double tagging` they can be classified as `switch spoofing` attack. Attacks will be performed with the l2 attacking tool `yersinia` [4]


:::info
1. Switch spoofing
:::

Switch spoofing attack requires DTP to be enabled on the port of attacked interface, that is why only cisco devices are vulnerable to this type of attack. Or devices that are able to enable DTP receiving [5]

The attacker sends a DTP packet to the interface of the Cisco device. By default, all interfaces are configured in the `dynamic auto` mode, and able to accept DTP packets. After a successful attack, links are set in the `trunk` mode, and hackers can access all VLANs in the network [6]

:::info
2. Double tagging
:::

Double tagging is a way to exploit the VLAN network with the additional tag on the ethernet frame. So, instead of one tag, there are two VLAN tags, one from the VLAN network of the attacker and another from the network of the victim.[7] This type of attack also requires some particular network configuration: there has to be one `native trunk` connection. The first VLAN id removes from the switch then transported to the destination switch, which sees the "wrong" tag of another network, so now the attacker can have access to another network. Hove ever, because users cant use the double tag technique, it is only a one-directional attack. [8]

#### Topology

In order for us to observe how does, above explained, methods of attacks work, we have built several topologies, Two basic, are shown in the picture below. Every other stand will look similar but have a different switch vendor.

For both of topologies IP addresses are chosen:
```
192.168.1.11 - user, vlan 100
192.168.1.12 - user, vlan 200
192.168.1.100 - attacker, vlan 100
```

For the first topology interfaces of switches, that are connected to the users are only added to the VLAN, so `administrative mode` is set to `dynamic auto` or interface can receive `DTP`

For the second Links to the users are configured in `access` mode, but the link between switches is configured in native trunk mode.

<center>

![](https://i.imgur.com/IlRfg9P.png)
Picture 1. Topology №1 used for switch spoofing
    
![](https://i.imgur.com/roQLYjo.png)
Picture 2. Topology №2 used for the double tagging
    
</center>

In the next sections, we tell more about particular configurations depending on the vendor of devices

### Task 3: Cisco

:::info
1. Switch Hopping
:::

As proof of work [9] of our network we provide a screenshot of the attacker console window, it is possible to ping `vlan100` user (same VLAN), and not possible to ping `vlan200` user (different VLAN) from the attacker machine

<center>
    
![](https://i.imgur.com/wXVqXEb.png)
    Picture 3. Proof of work
</center>

Now we are ready to implement Switch Hopping attack with `yersinia dtp -attack=1`, in the three pictures below you can see the status of the cisco interfaces before the attack, the attack itself, and the status of the cisco interfaces after the attack

<center>

![](https://i.imgur.com/G7aOmwq.png)
Picture 4. interfaces before attack
    
![](https://i.imgur.com/6xBLB9d.png)
Picture 5. yersinia attack
    
![](https://i.imgur.com/qD8jnh2.png)
Picture 6. interfaces after attack
    
</center>

As you can notice interface `Gi0/3` VLAN status has changed from 100 to `trunk`
In the picture below you can see the difference between attacker and user interfaces, first one has access to the other VLANs

<center>

![](https://i.imgur.com/ijqBZgC.png)
   Picture 7. information about `Gi0/3` interface
    
![](https://i.imgur.com/azdmjXV.png)
   Picture 8. information about `Gi0/1` and `Gi0/2` interface
</center>

After we have access to all VLANs, we can become full-fledged users of another VLAN. To this end, we need to create an interface that is configured to communicate to a particular VLAN.

```
sudo modprobe 8021q                        # use 8021q module
sudo vconfig add ens4 200                  # add interface connected with the vlan
sudo ifconfig ens4.200 up                  # turn this interface on
sudo ifconfig ens4.200 192.168.1.101 up    # add ip
```

After the interface is ready to work, you can see in the picture below, we can ping the user from the VLAN 200, as is shown in the picture below

<center>
    
![](https://i.imgur.com/BiTsGgD.png)
Picture 9. New interface
    
![](https://i.imgur.com/Q7eglBl.png)
Picture 10. Ping to the user in vlan 200 from the vlan 100
    
</center>

:::spoiler
1.    enable
2.    configure terminal
3.    vlan vlan-id
4.    name vlan-name
5.    mtu mtu-size
6.    remote-span
7.    end
8.    show vlan {name vlan-name | id vlan-id}
9.    copy running-config startup-config 

add vlan
1.    configure terminal
2.    interface interface-id
3.    switchport mode `dynamic auto` (preconfigured) or `desirable` for 3
most of the web sites suggest to use `mode access` , because it is not working with it
5.    switchport access vlan vlan-id
6.    end
7.    show running-config interface interface-id
8.    show interfaces interface-id switchport 

https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960x/software/15-2_2_e/vlan/configuration_guide/b_vlan_1522e_2960x_cg/b_vlan_152ex_2960-x_cg_chapter_011.html

:::

:::info
2. Double tagging
:::

After configuring network topology for this type of attack, we can see the status of the interface on the Cisco switch, as it was said previously one link is configured in the native trunk state. You can see screenshots with the status on the picture below.

<center>

![](https://i.imgur.com/UF7Ul22.png)
Picture 11. user side switch
    
![](https://i.imgur.com/r3156bN.png)
Picture 12. attacker side switch
    
</center>

Firstly, we will show how the attacker can ping users with the IP address 192.168.1.11 because they are in the same Vlan, you can see proof of that in the 3 pictures below.

```
sudo yersinia dot1q -attack 1 -source 0E:5C:49:19:32:BF -dest FF:FF:FF:FF:FF:FF -vlan1 0200 -priority1 07 -cfi1 0 -l2proto1 800 -vlan2 0300 -priority2 07 -cfi2 0 -l2proto2 800 -ipsource 192.168.1.100 -ipdest 192.168.1.12 -ipproto 1 -payload YERSINIA -interface ens3
```

<center>
    
![](https://i.imgur.com/dfk9THK.png)
Picture 13. From switch 2 to the attacker    
    
![](https://i.imgur.com/n7AsfZR.png)
Picture 14. From switch 2 to switch 1
    
![](https://i.imgur.com/T5qsJiy.png)
Picture 15. From switch 1 to the user of VLAN 100
    
</center>

Second, we will send a double-tagged packet to the user of the VLAN 200,

<center>

![](https://i.imgur.com/Tl067dw.png)
Picture 16. From switch 2 to the attacker
    
![](https://i.imgur.com/PRf7CMP.png)
Picture 17. From switch 2 to switch 1
    
![](https://i.imgur.com/Vu1TYef.png)
Picture 18. From switch 1 to the user of VLAN 200
    
</center>

As we can see in the pictures above, the packet has successfully traveled to the not intended VLAN 200, which means we have successfully bypassed VLAN restrictions with the double-tagged packet. But only in the one direction

:::spoiler
create vlan
access to all host pc vlan 100 200
trunk native to between switches, so vlan 300

#switchport trunk native vlan 300
#switchport trunk encapsulation dot1q

sudo yersinia dot1q -attack 1 -source 0E:5C:49:19:32:BF -dest FF:FF:FF:FF:FF:FF -vlan1 0200 -priority1 07 -cfi1 0 -l2proto1 800 -vlan2 0300 -priority2 07 -cfi2 0 -l2proto2 800 -ipsource 192.168.1.100 -ipdest 192.168.1.12 -ipproto 1 -payload YERSINIA -interface ens3
:::

### Task 4: Cumulus

:::info
2. Double tagging
:::

Because Cumulus is a full-featured Linux operating system for the networking industry created by Nvidia [10], and does not support some of the cisco inventions like `DTP`, it is possible to perform only one type of attack, Double tagging

First, we can see that in the picture below, the attacker isn't able to reach VLAN that he is not intended to.
The second attacker, on the same screenshot, is creating a packet with two tags

<center>
    
![](https://i.imgur.com/Brp8aoS.png)
Picture 19.  Attacker ping user from VLAN 200 and crafts packets
    
![](https://i.imgur.com/SoABa7c.png)
    Picture 20. From the attacker to switch 2
    
![](https://i.imgur.com/Bmtg0yf.png)
    Picture 21.  from switch 2 to switch 1
    
![](https://i.imgur.com/Y8SN24G.png)
    Picture 22.  from the switch 1 to user of VLAN 200

</center>

As we can see in the pictures above, the packet has successfully traveled to the not intended VLAN 200, which mean we have successfully bypassed VLAN restrictions with the double-tagged packet



:::spoiler

net show configuration

add br and ports to it
net add bridge bridge ports swp(n)

vlans id
net add bridge bridge vids (n)

add naative vlan
net add bridge bridge pvid 1

net add vlan (n) ip address

net del interface swp1 bridge access 100

sudo yersinia dot1q -attack 1 -vlan1 0200 -priority1 00 -cfi1 0 -l2proto1 800 -vlan2 0100 -priority2 00 -cfi2 0 -l2proto2 800 -ipsource 192.168.1.100 -ipdest 192.168.1.12 -ipproto 1 -payload YERSINIA -interface ens3
https://img-en.fs.com/file/user_manual/cumulus-linux-user-guide.pdf
:::

### Task 5: OpenvSwitch

:::info
1. Switch Hopping
:::

According to official documentation, it is possible to configure an open switch with the ability to receive DTP packets, with the command line `ovs-vsctl set bridge ovs-br0 other-config:forward-bpdu=true`. However, I could successfully implement this. But, since it is a possible option in the OpenvSwitch configuration it's worth mentioning this possibility. [11]

:::info
2. Double tagging
:::

OpenvSwitch is also Linux based system, that, except for some peculiarities, will be configured similarly to the `Cumulus` device[12]

<center>
    
![](https://i.imgur.com/yrEyFcZ.png)
Picture 23. From the attacker to switch 2
    
![](https://i.imgur.com/2O1sLiZ.png)
Picture 24.  from switch 2 to switch 1
    
![](https://i.imgur.com/G30gs8Q.png)
Picture 25.  from the switch 1 to user of VLAN 200
    
</center>

As we can see in the pictures above, the packet has successfully traveled to the not intended VLAN 200, which mean we have successfully bypassed VLAN restrictions with the double-tagged packet

:::spoiler

2
sudo yersinia dot1q -attack 1 -vlan1 0200 -priority1 00 -cfi1 0 -l2proto1 800 -vlan2 0100 -priority2 00 -cfi2 0 -l2proto2 800 -ipsource 192.168.1.100 -ipdest 192.168.1.12 -ipproto 1 -payload YERSINIA -interface ens3

ovs-vsctl show
ovs-vsctl list Bridge
ovs-vsctl list port eth3

ovs-vsctl del-br br0
ovs-vsctl add-br ovs-br0
ovs-vsctl set port ovs-br0 tag=200
ovs-vsctl add-port ovs-br0 eth1
ovs-vsctl add-port ovs-br0 eth2
ovs-vsctl add-port ovs-br0 eth3
ovs-vsctl set port eth1 tag=100
ovs-vsctl set port eth2 tag=200
of
ovs-vsctl set port eth3 vlan_mode=(there 4 of them like native taggeg\non etc)

ovs-vsctl set port eth3 trunks=100,200
ovs-vsctl del-port ovs-br0 eth3


ifconfig eth0 0
ifconfig ovs-br0 10.0.7.1/24 up
ip r add default via 10.0.7.254

ovs-vsctl del-br br0
ovs-vsctl del-port ovs-br0 eth3
ovs-vsctl clear port ovs-br0 tag
https://habr.com/ru/post/242741/
:::

## Security measures

As we have seen in the above sections it is possible to exploit different switch vendors, so it is logical to come up with security measures in order to prevent such types of attacks

:::info
1. Switch Hopping
:::

Because of the specific requirements of this type of attack, it is easier to defend against it

Cisco: put the interfaces into the `nonegotiate` or `access` mode.
OpenvSwitch: option `forward-bpdu` should be turned off, or configured in full understanding of consequences. And happily, it turned off by default
```
ovs-vsctl set bridge ovs-br0 other-config:forward-bpdu=false
```
Cumulus: does not support DTP

:::info
2. Double tagging
:::

This attack requires native VLANs to be configured and used in order to successfully send a double-tagged packet

Cisco:
set interfaces of the switch to `access` mode. 
`switchport access vlan (id)`

IF it necessary to use native use it in a separate VLAN
`switchport trunk native VLAN (id)`

All trunk ports have to the tagged explicitly (encapsulation is turned on)
`vlan dot1q tag native`

Since both of the switch solutions are Linux-based, defending mechanism against this type of attack is similar, also, logically they are similar to the cisco one

OpenvSwitch:
refrain to use native trunk
`ovs-vsctl set port (port) vlan_mode=(access or native taggeg)`


Cumulus:
refrain to use the native trunk
`net add bridge (name) pvid 0 (native vlan)`

Also, it is possible to implement a filter that refuses to receive packets with the two tags, but in that way, you forbid yourself to use double VLAN technology

## Results

Attacks were successfully performed in the virtual environment, was shown that it is possible to perform double tagging attack to switch of various vendors, switch hopping, on the other side is the attack inherent only to cisco devices, or devices with the cisco protocol compatibility

Methods presented to protect against types of attacks. They are logically similar to each other, without the dependency of the vendor, however, the basic configuration of the switch is dependent, that is why steps performed for the different vendor is different. For the cisco devices, it has to be disabled dynamic mode, for the Linux based, they have to be assigned in VLANs.

## References

[1] http://www.ttcenter.ir/ArticleFiles/ENARTICLE/3436.pdf
[2] https://ronnybull.com/assets/docs/bullrl-defcon24.pdf
[3] https://www.ietf.org/rfc/rfc3069.txt
[4] https://github.com/tomac/yersinia
[5] https://www.cvedetails.com/cve/CVE-1999-1129/
[6] https://www.securitylab.ru/analytics/495927.php
[7] https://www.cvedetails.com/cve/CVE-2005-4440/
[8] https://codeby.net/threads/ispolzovanie-vlan-double-tagging.73847/
[9] https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960x/software/15-2_2_e/vlan/configuration_guide/b_vlan_1522e_2960x_cg/b_vlan_152ex_2960-x_cg_chapter_011.html
[10] https://docs.nvidia.com/networking-ethernet-software/cumulus-linux-43/
[11] https://www.mankier.com/5/ovs-vswitchd.conf.db#Bridge_TABLE
[12] https://habr.com/ru/post/242741/