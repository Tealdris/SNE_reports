###### tags: `SSN`
:::success
# SSN Lab 7 Assignment: Firewall & Intrusion Detection Systems
**Name: Radomskiy Ilia**
:::

---

## Task 1: Configure topology, Select  and install IDS/IPS

:::info
snort
:::

For this lab I will be setting up Snort IDS/IPS on an ubuntu machine, to this end I have built the topology, you can see it in the picture below.

<center>

![](https://i.imgur.com/ERcaBgX.png)
Topology
</center>

In this hidden section, Installation steps are listed, they are provided for personal convenience

:::spoiler
[](https://www.youtube.com/watch?v=enSll_9Bjag)

[How to configure snort as IPS ](https://sublimerobots.com/2016/02/snort-ips-inline-mode-on-ubuntu/)

[How to configure snort as IPS - Habr](https://habr.com/ru/post/123474/)
[Snort official  web site](https://snort.org)

Install dependencies for `snort`
```
sudo apt-get install libdnet build-essential  bison flex  libpcap-dev  libpcre3-dev  libnet1-dev  zlib1g-dev  libnetfilter-queue-dev # daq: nfq  libmnl-dev libnfnetlink-dev libnetfilter_queue-dev
```

Install `snort`

```
sudo apt-get install snort
```

Downloading and configuring `daq` and `libdnet`

```
wget https://snort.org/downloads/snort/daq-2.0.7.tar.gz
sudo ./configure && sudo make && sudo make install
wget https://github.com/ofalk/libdnet/archive/refs/heads/master.zip
sudo ./configure && sudo make && sudo make install
```

This `snort.conf` file contains the main configuration of `snort`, and I will edit it in order to make `snort` work as IPS 

```
sudo nano /etc/snort/snort.conf
    ipvar HOME_NET localnetwork
RULE_PATH /etc/snort/rules/local.rules

config daq:afpacket
config daq_mode:inline
config policy_mode:inline
```

This line is used to start snort:
```
sudo snort -Q -c /etc/snort/snort.conf -i ens3:ens4 -A console
```
-Q for `inline` mode
-c for the configuration file
-i interface
-A set alert mode, print it in console

if you have registered on the `snort` web site, you can use rules made by other users with this command
```
wget https://www.snort.org/rules/snortrules-snapshot-2983.tar.gz?oinkcode=<your oink code goes here> -O snortrules-snapshot-2983.tar.gz
```
:::

In the next section, `iptables` rules are provided. They are used in order to redirect traffic from one interface on the ubuntu machine to another interface, that leads to the server

:::spoiler

Those rules are used in order to check `iptables` rules
```
sudo iptables -t nat -v -L POSTROUTING -n --line-number
sudo iptables -v -L FORWARD -n --line-number
sudo iptables -D FORWARD 1
```

This rule is needed for the traffic forwarding in the interfaces, so to `snort` machines act as a switch, they also can be edited in the `/etc/sysctl.conf` file.

```
sysctl -w net.ipv4.conf.all.forwarding=1
```

All rules below is used for `nat` configuration
```
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 192.168.122.225

sudo iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE
sudo iptables -A FORWARD -i ens3 -o ens4 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i ens3 -o ens4 -j ACCEPT

iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 192.168.122.225
sudo iptables -t nat -A POSTROUTING -s 192.168.122.0/24 -j SNAT --to-source 192.168.1.1
```

[Make a machine a NAT router](https://askubuntu.com/questions/988218/make-a-machine-a-nat-router)
[NAT in Ubuntu](https://ixnfo.com/nastroyka-maskaradinga-v-ubuntu.html)
:::


---

## Task 2: Working with IDS/IPS

:::info
1. Create a policy/rule to block all incoming ping packets.
:::

I wrote this rule to the file `local.rules`, in order to implement ICMP ping blocking
```
reject icmp any any <> any any (msg:"ICMP test"; sid:10000001; rev:001;)
```

:::info
2. Verify that the newly created rule/policy can correctly detect and block incoming ping packets from the attacker machines.
:::

Below you can see the group of screenshots:
In this, the attacker is making two `ping` session, on the first one `ping` are successful, on the second they are not because of rules implemented in `snort`

![](https://i.imgur.com/96zjnDa.png)

In this screenshot, you can find the output console of the `snort`, and we can see that the `ping` is have been blocked

![](https://i.imgur.com/MaghDV0.png)

For this one, I have captured Wireshark interception, so it is visible now, pings are going only in one direction - attacker receives back `destination unreachable`

![](https://i.imgur.com/gRMuGp5.png)

:::info
3. Write 5 more rules/policies of your own that can detect different attacks (possible attacks). 
:::

In the list below, there are some rules are provided for the `snort` to react to the different attack methods:


1. Attack: DOS
Attack command: `sudo hping3 -S -p 80 --faster --rand-source `192.168.122.228
Rule: block 𝚝𝚌𝚙 𝚊𝚗𝚢 𝚊𝚗𝚢 -> $𝙷𝙾𝙼𝙴𝙽𝙴𝚃 𝟾𝟶 (𝚏𝚕𝚊𝚐𝚜: 𝚂; 𝚖𝚜𝚐:"𝙿𝚘𝚜𝚜𝚒𝚋𝚕𝚎 𝙳𝚘𝚂 𝙰𝚝𝚝𝚊𝚌𝚔 𝚃𝚢𝚙𝚎 : 𝚂𝚈𝙽 𝚏𝚕𝚘𝚘𝚍"; 𝚏𝚕𝚘𝚠:𝚜𝚝𝚊𝚝𝚎𝚕𝚎𝚜𝚜; 𝚜𝚒𝚍:𝟹; detection_filter: 𝚝𝚛𝚊𝚌𝚔 𝚋𝚢_𝚍𝚜𝚝, 𝚌𝚘𝚞𝚗𝚝 𝟸𝟶, 𝚜𝚎𝚌𝚘𝚗𝚍𝚜 𝟷𝟶;)


2. Attack: Slowloris
Attack command: `sudo slowloris 192.168.122.228`
Rule: block tcp any any -> any 80 (msg:"Possible Slowloris Attack Detected";flow:to_server,established; pcre:"/X-a|3a| \d{4}../"; sid:10000005;)

3. Attack: SSH
Attack command: `ssh ubuntu@192.168.122.228`
or 
Attack command:`nmap 192.168.122.228`
Rule: drop tcp any any -> $HOME_NET 22 (msg:"SSH connection attempt"; sid:1000003; rev:1;)

4. Attack: large pings
Attack command: `ping -s 65507 192.168.122.228`
Rule: drop icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"ICMP Large ICMP Packet"; dsize:>800; reference:arachnids,246; classtype:bad-unknown; sid:499; rev:4;)

5. Attack: SQL injection
Attack command: we will go to the `192.168.122.228/?1=1` web page instead of `192.168.122.228`
Rule: drop tcp any any -> any any (msg: "SQL Injection"; content: "GET"; http_method; uricontent: "?1=1"; nocase; sid:3000001; rev:1;)



:::info
4. Show the new 5 rules and explain how the attacks would work and how the rule/policy would be able to detect it.
:::

All rules as a part of the `local.rules` file are shown in the picture below:

![](https://i.imgur.com/1uBYZp5.png)

1. Attack: DOS
explanation: DOS denial of service attack is performed by sending a lot of the packets, in order to shut down the server, because it would be not able to handle all of them
So in rule, we may work with the number of packets per sometimes
Rule: block 𝚝𝚌𝚙 𝚊𝚗𝚢 𝚊𝚗𝚢 -> $𝙷𝙾𝙼𝙴𝙽𝙴𝚃 𝟾𝟶 (𝚏𝚕𝚊𝚐𝚜: 𝚂; 𝚖𝚜𝚐:"𝙿𝚘𝚜𝚜𝚒𝚋𝚕𝚎 𝙳𝚘𝚂 𝙰𝚝𝚝𝚊𝚌𝚔 𝚃𝚢𝚙𝚎 : 𝚂𝚈𝙽 𝚏𝚕𝚘𝚘𝚍"; 𝚏𝚕𝚘𝚠:𝚜𝚝𝚊𝚝𝚎𝚕𝚎𝚜𝚜; 𝚜𝚒𝚍:𝟹; detection_filter: 𝚝𝚛𝚊𝚌𝚔 𝚋𝚢_𝚍𝚜𝚝, 𝚌𝚘𝚞𝚗𝚝 𝟸𝟶, 𝚜𝚎𝚌𝚘𝚗𝚍𝚜 𝟷𝟶;)

2. Attack: Slowloris
explanation: Slowloris is a DOS attack, but different, in this method attacker keep as many connections to the target web server open as long as possible
So we can work with established connection and signature of the `Slowloris`
Rule: block tcp any any -> any 80 (msg:"Possible Slowloris Attack Detected";flow:to_server,established; pcre:"/X-a|3a| \d{4}../"; sid:10000005;)

3. Attack: SSH
explanation: can be used as if the attacker want to connect to the server via `ssh` or as part of the scanning process `nnmap`
We can look after SSH port, 22
Rule: drop tcp any any -> $HOME_NET 22 (msg:"SSH connection attempt"; sid:1000003; rev:1;)

4. Attack: large pings
explanation: In this case we just inspect size of the ICMP packet
Rule: drop icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"ICMP Large ICMP Packet"; dsize:>800; reference:arachnids,246; classtype:bad-unknown; sid:499; rev:4;)

5. Attack: SQL injection
explanation: SQL injection occures when attacker try to use not appropriate characters in order to  get access to the web server
So we can detect `get` headers in order to prevent this attack
Rule: drop tcp any any -> any any (msg: "SQL Injection"; content: "GET"; http_method; uricontent: "?1=1"; nocase; sid:3000001; rev:1;)


:::info
5. Describe how you triggered the alert for all newly created rules.
:::

Triggers to alert for `snort` are provided below:

1. Attack: DOS
detection_filter: 𝚝𝚛𝚊𝚌𝚔 𝚋𝚢_𝚍𝚜𝚝, 𝚌𝚘𝚞𝚗𝚝 𝟸𝟶, 𝚜𝚎𝚌𝚘𝚗𝚍𝚜 𝟷𝟶; will coun packets per second on the destination

2. Attack: Slowloris
pcre:"/X-a|3a| \d{4}../"; Perl Compatible Regular Expressions stands for the Slowloris attack

3. Attack: SSH
22; port 

4. Attack: large pings
dsize:>800; size

5. Attack: SQL injection
flow:to_server,established; content "?1=1";

:::info
6. Attach the alert itself for each given rule/policy to the report in readable text. 
:::

Attacks:

1. Attack: DOS
Attack command: `sudo hping3 -S -p 80 --faster --rand-source `192.168.122.228
Rule: block 𝚝𝚌𝚙 𝚊𝚗𝚢 𝚊𝚗𝚢 -> $𝙷𝙾𝙼𝙴𝙽𝙴𝚃 𝟾𝟶 (𝚏𝚕𝚊𝚐𝚜: 𝚂; 𝚖𝚜𝚐:"𝙿𝚘𝚜𝚜𝚒𝚋𝚕𝚎 𝙳𝚘𝚂 𝙰𝚝𝚝𝚊𝚌𝚔 𝚃𝚢𝚙𝚎 : 𝚂𝚈𝙽 𝚏𝚕𝚘𝚘𝚍"; 𝚏𝚕𝚘𝚠:𝚜𝚝𝚊𝚝𝚎𝚕𝚎𝚜𝚜; 𝚜𝚒𝚍:𝟹; detection_filter: 𝚝𝚛𝚊𝚌𝚔 𝚋𝚢_𝚍𝚜𝚝, 𝚌𝚘𝚞𝚗𝚝 𝟸𝟶, 𝚜𝚎𝚌𝚘𝚗𝚍𝚜 𝟷𝟶;)

In the pictures below you may see:

Hacker is attempting `DOS` attack

![](https://i.imgur.com/rppqO7U.png)

`Wireshark` capture or packet rejection

![](https://i.imgur.com/lot5UD1.png)

`Snort` console output of packet rejection

![](https://i.imgur.com/yOYSy3M.png)

2. Attack: Slowloris
Attack command: `sudo slowloris 192.168.122.228`
Rule: block tcp any any -> any 80 (msg:"Possible Slowloris Attack Detected";flow:to_server,established; pcre:"/X-a|3a| \d{4}../"; sid:10000005;)


In the pictures below you may see:

Hacker is attempting `Slowloris DOS` attack

![](https://i.imgur.com/aXOMZvm.png)

`Wireshark` capture or packet rejection

![](https://i.imgur.com/e5gCbmG.png)

`Snort` console output of packet rejection

![](https://i.imgur.com/qYL9qbe.png)


3. Attack: SSH
Attack command: `ssh ubuntu@192.168.122.228`
or 
Attack command:`nmap 192.168.122.228`
Rule: drop tcp any any -> $HOME_NET 22 (msg:"SSH connection attempt"; sid:1000003; rev:1;)

In the pictures below you may see:

Hacker is attempting `SSH` connection or investigation

![](https://i.imgur.com/oVrW6We.png)

`Wireshark` capture or packet rejection

![](https://i.imgur.com/jH1uYhL.png)

`Snort` console output of packet rejection

![](https://i.imgur.com/byVrhXE.png)


4. Attack: large pings
Attack command: `ping -s 65507 192.168.122.228`
Rule: drop icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"ICMP Large ICMP Packet"; dsize:>800; reference:arachnids,246; classtype:bad-unknown; sid:499; rev:4;)

In the pictures below you may see:

Hacker is attempting `large ping` attack

![](https://i.imgur.com/BEj1yzP.png)

`Wireshark` capture of packet rejection

![](https://i.imgur.com/DfuppzA.png)

`Snort` console output of packet rejection

![](https://i.imgur.com/118UKlj.png)


5. Attack: SQL injection
Attack command: we will go to the `192.168.122.228/?1=1` web page instead of `192.168.122.228`
Rule: drop tcp any any -> any any (msg: "SQL Injection"; content: "GET"; http_method; uricontent: "?1=1"; nocase; sid:3000001; rev:1;)

In the pictures below you may see:

Main Sign window, the attacker will try to enter `192.168.122.228/?1=1` instead of `192.168.122.228` in order to implement SQL injection

![](https://i.imgur.com/nBafy3y.png)

`Burpsuite` intercepted traffick of request, `GET` header is `192.168.122.228/?1=1`

![](https://i.imgur.com/ZKbAP0e.png)

`Wireshark` capture of packet rejection

![](https://i.imgur.com/ISvO69Q.png)

`Snort` console output of packet rejection

![](https://i.imgur.com/vzF61WH.png)

The error that we are getting the trying this attackk

![](https://i.imgur.com/FEHNNKt.png)

:::info
7. Create one rule/policy which would be triggered if a server vm / IDS/IPS vm would browse to microsoft.com website.
:::

In order to block access to `microsoft.com` for the user rule were added to the `local.rules`

```
reject tcp any any <> any $HTTP_PORTS (msg:"User is searching for ms.com"; content:"microsoft.com"; nocase; sid:991999;)
```

In the pictures below you may see:

The user is trying to `curl microsoft.com` but haven't succeeded, in comparison, you can observe a successful `curl` attempt, below, right after the first try

![](https://i.imgur.com/gStV8j9.png)

`Snort` console output of packet rejection

![](https://i.imgur.com/YKzXEnp.png)

---

## Task 3: Answer the following questions

:::info
1. What is a zero-day attack?
:::

zero-day attack, a term that is used to describe vulnerability that hasn't been found or resolved, or software against that we have no counter defense mechanism developed yet

:::info
2. Can IDS/IPS detect zero-day attack? if yes why? if no why?
:::

Yes, this type of attack can be detected with IPS/IDS if the attack method is similar to another one - well known (almost identical signature) 
Because there is [Proactive cyber defense](https://en.wikipedia.org/wiki/Proactive_cyber_defence), in this term included sandboxes, behavior analysis, code emulation, and Heuristic analysis, all of them serve to detect previously unknown computer viruses.
Modern antiviruses use a combination of Proactive cyber defense and defense against the already known attacks/viruses


:::info
3. Given a network that has 1 million connections daily where 0.1% are attacks. If the IDS has a true positive rate of 95% what false positive alarm do we need to achieve to ensure the probability of an attack, given an alarm is 95%?
:::

There are 1 million connections daily, where 0.1% of them are attacks, so 1000/day. From the other perspective, amount of connections that weren't refused is 999 000/day. Amount of true positive connections is 950 (1000x95%). After that, we assume that the false-positive rate is equal to:

x means multiplication

**999000**x**y**/**100**. 

If we want to count the probability of false-positive alarm we need to count 

950/(950+(**999000**x**y**/**100**))

After that we are close to final equation , we substract previous on from 100 and equate our system to 95%, so alarm rate:

0.100 - 950/(950+(**999000**x**y**/**100**)) = 0.95

From that, we are able to find **y**, the answer located in the table below

**Table with the final results**
|  Amount    | Percentage | Explanation                                |
| --------  | --------   | --------                                   |
| 1 000 000 | 100        | connections a day                          |
|     1 000 |     0.1    | attack a day                               |
|   999 000 |  99.9      | connections passed through                 |
|       950 |  95        | True positive                              |
|           |   1.806    | Final answer / % of false positive alarm   |

In the hidden part below there are installations for Suricata. Attempt was unsuccessful

