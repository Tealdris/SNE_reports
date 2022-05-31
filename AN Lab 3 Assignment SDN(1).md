###### tags: `AN`
:::success
# AN Lab 3 Assignment: SDN
**Name: Radomskiy Ilia**
:::

## Task 1: Preparation
:::info
a. Select an SDN controller that you would like to try. 
:::spoiler
For example (opendaylight, Ryu, faucet,
ONOS)
:::

I will be using a faucet in order to perform SDN in GNS3

:::info
b. GNS3 might already have a template for these controllers, try to use these templates as it will save you a lot of time and troubleshooting.
:::

The faucet already has a docker solution, that can be used inside gns3

![](https://i.imgur.com/M4Ve1NL.png)

[Faucet documentation](https://docs.faucet.nz/en/latest/intro.html)
[](https://cyberlab.pacific.edu/courses/comp177/labs/lab-12-openflow)

:::info
c. You can use OpenVSwitch as your virtual switching solution (check ovs-vsctl command)
:::

OpenvSwitch can be installed with Docker too

![](https://i.imgur.com/HDrnwoh.png)

[Open vSwitch documentation](https://docs.openvswitch.org/en/latest/intro/install/documentation/)

:::info
d. Try to draw a network scheme before you start the lab. This will help you in the deployment
phase.
:::

My topology looks like this

![](https://i.imgur.com/7z60Xdo.png)

---
## Task 2: Deployment

:::info
a. Try to redo one of the previous labs, but this time with SDN
:::

I will redo VLAN lab from INR
Next conditions of topology considered:

IP adresses have been set for:

    -Faucet: 192.168.122.10
    -openvswirches: 192.168.122.11-14
    -pcs: 10.20.*.*/24
    
Devices have been configured:

    -openvswirch:
    -- set number of briege to x (number of switch), set ip address of controller (faucet), delite management interface from briege,
    
        ovs-vsctl set bridge br0 other-config:datapath-id=000000000000000x \
            -- set bridge br0 fail_mode=secure \
            -- set-controller br0 tcp:192.168.122.10:6653 \
            -- set controller br0 connection-mode=out-of-band \
            -- --if-exists del-port eth0

The next configuration step takes place in the faucet, I will change the file `faucet.yaml` every time when I need to change the way how switches are working:

    -faucet: 
        --edited vi /etc/faucet/faucet.yaml file
        --checking check_faucet_config /etc/faucet/faucet.yaml
        --reloading configuration pkill -HUP ryu-manager
    
    
---
## Task 3: Verification

:::info
a. Show your SDN flows that are related to the previous lab that will redo
:::

When we apply our configuration we are getting this output in the terminal

![](https://i.imgur.com/dMRayU1.png)

In this particular case we can see, things that were configured by us previous

A little below, in the same output, we can find some parameters of the flow, like several types of metrics and packets arguments

![](https://i.imgur.com/rnvINji.png)

:::info
b. Try to explain ovs-ofctl dump-flows br0
what are the fields in each flow
:::

The flow of the Open vSwitch can be shown by typing: `ovs-ofctl dump-flows br0`.

![](https://i.imgur.com/9U1BD25.png)

This flow contains a little bit more information, such as `vlan_id`, IP and mac addresses, and action, thing what we should do with the packet, and others

After configuration, we can check if our topology is working or not

For example, in the picture below you can see pinging from pc 7 to pc5 and pc8, pc7 and pc5 are connected to the same switch but located in different VLANs

![](https://i.imgur.com/kITF5bu.png)

In this picture, I changed the IP address on pc5, so pc5 and pc7 are located in the same network but different VLANs

![](https://i.imgur.com/yQEhVlz.png)

In this case, I am doing ping from pc3 to pc1 (success for different switches but same VLAN), and pc3 and pc4 (fail for same switch but different VLAN)


![](https://i.imgur.com/d80zTMX.png)

As a result of setting up SDN, we got a solution that can configure a whole network from one device

---
## Things that was usefull in process (some useless now)

:::spoiler

**opendaylight**

installation
apt-get update
apt-get install -y bash-completion software-properties-common python-software-properties sudo curl ssh git
nano /etc/ssh/sshd_config  ###(change PermitRootLogin to yes)
service ssh start
ssh-keygen -t rsa -P ""
1: some command
(instead) sudo apt -y install openjdk-8-jdk

nano ~/.bashrc
export JAVA_HOME="/usr/lib/jvm/java-8-oracle"
. ~/.bashrc
echo $JAVA_HOME    ### Verification

wget https://nexus.opendaylight.org/content/groups/public/org/opendaylight/integration/distribution-karaf/0.3.0-Lithium/distribution-karaf-0.3.0-Lithium.tar.gz

ifconfig what is uor ip
passwd set ip for root

curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash - (there is a mistake but go on)
apt-get install nodejs
git clone https://github.com/CiscoDevNet/OpenDaylight-Openflow-App.git
**sed -i 's/localhost/<server_ip>/g' ./OpenDaylight-Openflow-App/ofm/src/common/config/env.module.js  ### Use the IP we noted earlier, for "server_ip"**

apt install npm

npm install -g grunt-cli

cd distribution-karaf-0.3.0-Lithium
sudo su
./bin/karaf
./bin/karaf clear

feature:install odl-restconf-all odl-openflowplugin-all odl-l2switch-all odl-mdsal-all odl-yangtools-common


ssh root@192.168.122.208
cd /home/ubuntu/OpenDaylight-Openflow-App/
grunt

**fix**
**1:** Some index files failed to download. They have been ignored, or old ones used instead.

sudo su 
cd /var/lib/apt/lists/
rm -fr *
cd /etc/apt/sources.list.d/
rm -fr *
cd /etc/apt
sudo cp sources.list sources.list.old
sudo cp sources.list sources.list.tmp
sed 's/ubuntuarchive.hnsdc.com/us.archive.ubuntu.com/' sources.list.tmp | sudo tee sources.list
sudo rm sources.list.tmp*
apt-get clean
apt-get update

ethernet type 35020
max len 65535
cookie 3098476543630901000

**2:** 
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null


**openvswitch commands:**

ovs-vsctl set bridge br0 stp_enable=true
ovs-vsctl set-controller br0 tcp:192.168.122.208:6633

ovs-vsctl show
ovs-ofctl show br0

ovs-vsctl set bridge br0 other-config:datapath-id=0000000000000004 \
            -- set bridge br0 fail_mode=secure \
            -- set-controller br0 tcp:192.168.122.10:6653 \
            -- set controller br0 connection-mode=out-of-band \
            -- --if-exists del-port eth0
            

**libs:**

feature:install odl-vtn-manager-neutron
feature:install odl-vtn-manager-rest
feature:install odl-mdsal-apidocs
feature:install odl-dlux-all

docker

ip addres add ip eth

sudo mn --controller remote --topo tree,depth=3
ryu-manager --verbose --observe-links ryu.topology.switches ryu.app.rest_topology ryu.app.ofctl_rest ryu.app.simple_switch
python3 ryu/ryu/gui/controller.py


sudo docker run -it --name faucetctl --network=host faucet/faucet /bin/bash
apt add nano
nano /usr/etc/faucet/faucet.yaml
ryu-manager --verbose faucet.faucet


version: 2
vlans:
    100:
        name: "clock"
        unicast_flood: True
        max_hosts: 3
acls:
    101:
        - rule
            dl_src: "mac"
            actions:
                allow: 0
        - rule:
            actions:
                allow: 1
dps:
    Open vSwitch:
        dp_id: 0x0000b6853c386d4e #dpid of switch
        hardware: "Open vSwitch"
        interfaces:
            1:
                native_vlan: 100
                name: "clock"
            2:
                native_vlan: 100
                name: "VLAN 2001"
                acl_in: 101
:::