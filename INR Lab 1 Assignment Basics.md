###### tags: `INR`

:::success
# INR Lab 1 Assignment: Basics
**Name: Radomskiy Ilia**
:::




## Task 1: Preparation
:::info
Read the notes, install GNS3 and dependencies
:::


You can simply install GNS3 by following the official manual [1].
It contains such commands as
```
sudo apt install gns3-gui gns3-server
sudo apt install docker-ce
ubridge libvirt kvm wireshark docker
sudo apt install qemu-kvm qemu

```
Picture of installed GNS3 you may see in the figure 1

<center>

![](https://i.imgur.com/WSD99Yk.png)

Figure 1: GNS3
</center>


---
## Task 2: Installation

:::info
Create a project and configure the pre-installed Ubuntu Cloud Guest template
:::


You can use a simple Linux machine inside GNS3. In order to do that you need to choose it from the new template window
You may see the result in figure 2

<center>

![](https://i.imgur.com/ksgCeu4.png)
Figure 2: Ubuntu Cloud Guest in GNS3

</center>

:::info
Find several ways to configure internet access in GNS3
:::
There are several ways to connect your environment to the internet:
Cloud - allows GNS3 machines to use our workstation interface. This is a built-in function of gns3 and you don't need to configure it

TAP interface - creates a bridge between GNS3 and our workstation interface. In order to establish TAP connections we inter lines below
Create bridge interface ```sudo brctl addbr br0```
Turn it up ```sudo ip l s dev br0 up```
Crete TAPo interface ```sudo tunctl -t tap0 -u st10```
Turn it up ```sudo ip l s dev tap0 up```
Connect two interfaces TAP and  br0 ```sudo brctl addif br0 tap0```
Connect bridge with our work station interface  ```sudo brctl addif br0 tap0```

NAT - gives to GNS3 machines virtual IP. And this is a built-in function of gns3 and you don't need to configure it

You can see different ways to connect the internet to the GNS3 in figure 

<center>

![](https://i.imgur.com/F70rmT9.png)
Figure 3: different ways to connect internet in GNS3
</center>


## Task 3: Switching

:::info
Create topology (figure 4)
:::

In order to make SSH connections, we should do some steps [2]
1) Install OpenSSH
```
sudo apt install openssh-server
```
2) Enable it
```
sudo systemctl enable sshd
```
For Nginx [3] steps are actually the same, but with changing 'openssh-server' to 'nginx'

:::info
Configure VMs with private static IPs under a /28 subnet
:::
Under /28 subnet mask, there are  16 addresses, but only 14 of them may be taken by PCs
To set up IP you need to change '50-cloud-init.yaml' file, by changing some values according to figure
```
sudo nano /etc/netplan/50-cloud-init.yaml
```

<center>

![](https://i.imgur.com/eZpg8FG.png)
Figure 5: 50-cloud-init.yaml configuration
</center>


We may check connectivity between our VMs by using ping, traceroute and other commands

For Nginx server checking we will be using wget. 
Proof of work of both servers you may see in a figure 5


<center>

![](https://i.imgur.com/GNz2F6I.png)

Figure 6.1: Reaching from admin to web nginx server with wget utility
</center>

<center>

![](https://i.imgur.com/iDAOJAM.png)

Figure 6.2: Reaching from web to admin via SSH
</center>

## Task 4: Routing

:::info
Change the topology
:::

For gateway we will be using Mikrotik Router

<center>

![](https://i.imgur.com/qMHJ6Uw.png)

Figure 6: New topology
</center>

From all of the possible ways of internet connection, we will use NAT connection. In figure 7 you may see the checking process of establishing a connection from the router to the internet

<center>

![](https://i.imgur.com/AmpZCDD.png)
Figure 7: Establishing connection

</center>

:::info
Configure port forwarding for HTTP and ssh to Web and Admin
:::

In order to get access to Web, Admin and Worker machines we will you port forwarding, with commands which are shown below. Those commands also can be used to port forward to 'admin' [4]
This line allows getting access from Ubuntu gest to the internet
```
chain=srcnat action=masquerade src-address=192.168.20.0/24
```

This line allows to get access to Ubuntu gest from the internet via port 8080, or in other words, send request to web page
```
chain=dstnat action=netmap to-addresses=192.168.20.2 to-ports=80 
      protocol=tcp in-interface=ether3 dst-port=8080
```   
This line allows sending a reply to the request from Ubuntu guest
```
chain=dstnat action=netmap to-addresses=192.168.20.2 to-ports=80 
      protocol=tcp src-address=192.168.20.0/24 dst-address=192.168.122.106 
      in-interface=ether3 dst-port=8080 
```

To get access to the admin PC via SSH we need to specify a different port 
```
chain=dstnat action=netmap to-addresses=192.168.20.3 to-ports=22 
      protocol=tcp src-address=192.168.20.0/24 dst-address=192.168.122.106 
      in-interface=ether3 dst-port=8083 
```
:::info
Configure port forwarding for HTTP and ssh to Web and Admin
:::

Everything set up before may be checked by entering nginx web page,  and connecting ssh between admin's PC, webs PC and our workstation. Results are shown in the figures below 

<center>

![](https://i.imgur.com/4uUDHYG.png)
Figure 8: Nginx web page
</center>

<center>

![](https://i.imgur.com/Nfe2wsZ.png)

Figure 9: SSH conection
</center>

Thus, as a result of this laboratory work, it was performed:
GNS3 installation and some of its dependencies
Was created two network topologies
On the first topology, we were checking basic commands for monitoring of network anв on network switching in general
On the second we were learning about network routing using the Mikrotik router.
In order to check everything is working, we entered the web page of Nginx server from our workstation, we established an SSH connection between our workstation


---
## References:

1: [GNS3 Linux Install](https://docs.gns3.com/docs/getting-started/installation/linux/)
2: [УСТАНОВКА SSH В UBUNTU](https://losst.ru/ustanovka-ssh-ubuntu-16-04)
3: [УСТАНОВКА NGINX UBUNTU](https://losst.ru/ustanovka-nginx-ubuntu-16-04)
4: [Проброс портов в роутере MikroTik](https://www.technotrade.com.ua/Articles/mikrotik-port-forwarding.php)


:::spoiler
sudo brctl addbr br0
sudo ip l s dev br0 up
sudo tunctl -t tap0 -u st10
sudo ip l s dev tap0 up
sudo brctl addif br0 tap0
sudo brctl addif br0 eno1
sudo ifconfig eno1 0.0.0.0 promisc 
sudo ip a a 10.1.1.67/24 dev br0

:::