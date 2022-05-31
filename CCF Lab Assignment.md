###### tags: `CCF`
:::success
# CCF Lab Assignment
**Name: Ilia Radomskiy, Alisher Arystay, Polina Fige**
:::

# The adventure of Evil Morty in the search of Szechuan sauce

### Narrator

Once upon a time, there was a boy called Evil Morty. Yes, Evil Morty, the evil version of Morty from our realm. He was jealous of not being able to try a notorious Szechuan sauce. Moreover, he didn't like his granddad Rick making fun of him because of this.

Evil Morty was a very smart guy, and his acute mind allowed him to come up with a plan to get his hands on the recipe of this Szechuan sauce and get his revenge from his grandfather Rick. For his plan to come true, Evil Morty needed to get an access to the Smiths' personal data, because only there Rick could've kept this overly valuable asset.

### Council of Ricks (1st viewpoint)

<center>

![](https://i.imgur.com/0mEy7Hp.png)

</center>

The time zone for machines is configured to ***Pacific Standard time***. That's why all other incident related stuff is changed to this time zone.

<center>

![](https://i.imgur.com/3GuTip0.png)

</center>

The first action the attacker took was making a small investigation on the availability of connection to the Smiths' server. We, the proud members of the Council of Ricks, may assume this from `case001.pcap` file that wrote down all the packet transfer from and to the server and image and log files from machines. There we found that someone unknown tried to ***nmap scan*** the server from the outside to know if he can reach and get a response to his request.

The attacker ***scanned open ports via nmap at 02:19:13 September 19, 2020*** and ***the server with IP 10.42.85.10*** positively replied to it.

<center>

![](https://i.imgur.com/P0Jb0tR.png)
    
Local to UTC-6 = 02:19:13

</center>

Then, after seeing that he was able to reach the server, he decided to enter the system hard way - he brute forced his way to the sacred storage.

***The attacker started brute forcing at 02:19:26***.

<center>
    
![](https://i.imgur.com/350GxNp.png)
    
Local to UTC-6 = 02:19:26

![](https://i.imgur.com/rPFm0Ph.png)

Local time to UTC-6 = 02:21:28
    
</center>

We, the proud members of the Council of Ricks, can be sure of the fact that the attacker was successful in his attempt of getting an access to the system ***as an administrator from 194.61.24.102 ip address via port 3389 at 02:21:48*** after numerous tries through some initially unidentified password-cracking software. He used a pretty common way of attacking ***open RDP port 3389***.

<center>

![](https://i.imgur.com/7jrlMp4.png)
    
UTC-6 = 2:21:48
    
</center>

### Evil Morty (1st viewpoint)

Hmm, C137's Rick... I guess I'm going the right way.

<center>

![](https://i.imgur.com/Aw9i4ET.png)

</center>

### Council of Ricks (1st viewpoint)

The server is ***Windows Server 2012 R2 build 9600***.

<center>

![](https://i.imgur.com/Udz0vyC.png)
    
</center>

The ***IP address 194.61.24.102*** was analyzed and it ***was identified as a malicious one***.

<center>
    
![](https://i.imgur.com/AemfaBj.png)

![](https://i.imgur.com/UVpKNMA.png)

194.61.24.102 information

</center>

After ***logging in as an administrator*** to the system, the attacker immediately ***downloads a malicious file coreupdater.exe at 02:24:06 to the server***.

<center>

![](https://i.imgur.com/BRV36a1.png)

</center>

It was found out that the attacker ***used Internet Explorer 11 for downloading the malware from user agent: User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; rv:11.0) like Gecko***.

<center>

![](https://i.imgur.com/vVuw1xW.png)

</center>

***Coreupdater.exe*** was confirmed ***to call back to 203.78.103.109 ip address*** that was reported to be ***malicious***. By looking into its details, it was possible to identify malware to be of a ***trojan.metasploit*** type.

<center>

![](https://i.imgur.com/9Tm7wly.png)
    
sha256sum coreupdater.exe

![](https://i.imgur.com/PN2sVTh.png)
    
![](https://i.imgur.com/NPSNDPj.png)
    
Data about trojan from the sandbox
    
![](https://i.imgur.com/Kaew7WQ.png)

new ip 203.78.103.109 port 443
    
</center>

`coreupdater.exe` on the Server calls back to a new IP address: ***203.78.103.109:443 from Thailand***.

<center>

![](https://i.imgur.com/HiW2anO.png)
    
![](https://i.imgur.com/ZbCnbL1.png)

203.78.103.109 information
    
</center>

After downloading the malware, it ***was added to the registry keys*** by being ***executed via powershell*** for it ***to be started during the log in*** process.

<center>
    
![](https://i.imgur.com/W7QVI3L.png)
    
found in
Windows/System32/config/SOFTWARE/Microsoft/Windows/CurrentVersion/Run/Coreupdate

![](https://i.imgur.com/5uGJ6Sm.png)
    
coreupdater was executed from powershell and added to Registry keys to be executed on login
    
</center>

***Installed malware was captured in the list of autorun processes.***

<center>
    
![](https://i.imgur.com/W4GNSZl.png)

</center>

Right after that, several ***other processes were said to be malicious*** as well: ***spoolsv.exe was infected***.

<center>

![](https://i.imgur.com/GdiCvvz.png)
    
![](https://i.imgur.com/drJTHdR.png)

</center>

It is shown that after downloading the malware at 02:24:06, ***`spoolsv.exe` was active at 02:29:40***.

<center>

![](https://i.imgur.com/lpkJL3G.png)
    
Local time to UTC-6 = 02:29:40

</center>

It is assumed that ***`coreupdater.exe` metasploit performed an injection to local `spoolsv.exe`, because of its unusually large payload***. This huge payload might be some sort of script.

After it, `coreupdater.exe` process became a zombie-process with 0 threads active listed among other processes.

<center>

![](https://i.imgur.com/kyodCmk.png)

</center>

Since the attacker logged in to the system as an administrator, ***files he worked with after the session started were found***.

<center>
    
![](https://i.imgur.com/Gay87qR.png)

files administrator was interested in
/Users/Administrator/AppData/Roaming/Microsoft/Windows/Recent
</center>

***The attacker deleted `SECRET_beth.txt` file at 02:34:27***.

<center>

![](https://i.imgur.com/iC5SsKd.png)
    
'secret' folder was entered
    
![](https://i.imgur.com/csFH43u.png)

![](https://i.imgur.com/DDDXabB.png)

</center>

***Then the attacker created a new file called `Beth_Secret.txt` with modified content at 02:34:56***.

<center>

![](https://i.imgur.com/QJTcJVt.png)

</center>

It can be seen from web cache that ***the attacker accessed these files from Internet Explorer browser between 02:32 and 02:35***.

<center>

![](https://i.imgur.com/p0q0jX9.png)

IE history
    
![](https://i.imgur.com/8z7GMaN.jpg)

IE history in the .dat file

</center>

***The attacker created an archive file called `secret.zip` at 02:35*** probably to add files to it. The file was deleted, but traces to it like web cache and symbolical link to it were left. He, most probably, took the archive file out and deleted it afterwards.

<center>

![](https://i.imgur.com/rNuCAV9.png)
    
![](https://i.imgur.com/ybh7mrp.png)
    
</center>

After injecting the server with malicious file, it is seen that the attacker ***accessed Desktop Machine (10.42.85.115) from the server (10.42.85.10) via TCP port 3389 at 02:35:55***. The attacker ***used the same admin credentials*** he used to enter the server to access the Desktop PC.

<center>
    
![](https://i.imgur.com/hSJUk9v.png)
    
![](https://i.imgur.com/LcaiTSz.png)

</center>

### Evil Morty (1st viewpoint)

Hmm, Rick, didn't know that your big cousin's name was Bill...

<center>

![](https://i.imgur.com/sehp1vm.png)

</center>

### Council of Ricks (1st viewpoint)

***Parameters of desktop machine - Windows 10 Enterprise build 19041***

<center>

![](https://i.imgur.com/LNdBs7e.png)

</center>

After accessing the Desktop PC, the attacker followed the same pattern of actions. ***He downloaded the coreupdater.exe malware from 194.61.24.102 at 02:39:58*** once again.

<center>

![](https://i.imgur.com/UMF9n8s.png)

trojan.metasploit from 194.61.24.102 to Desktop PC
    
</center>

The attacker ***used Microsoft Edge browser*** to download the malware that was identified from ***user agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.19041***.

<center>

![](https://i.imgur.com/lUUKvBP.png)

</center>

***`coreupdater.exe` was executed from the powershell and was added to the registry keys for the same purpose***.

<center>

![](https://i.imgur.com/yJdoCue.png)
    
![](https://i.imgur.com/4vQDYjh.png)
    
</center>

***Installed malware was captured in the list of autorun processes.***

<center>

![](https://i.imgur.com/xC57QzQ.png)
    
</center>

`coreupdater.exe` on the Desktop PC calls back to a new IP address: ***203.78.103.109:443 from Thailand***.

<center>

![](https://i.imgur.com/sJmTxhv.png)

</center>

Since the attack method remains same, ***it will try to perform similar actions: execution, injection and transfer*** - regular behaviour for metasploit type trojan.

It can be noted that the beginning looks like `coreupdater.exe`.

<center>

![](https://i.imgur.com/deO7HcJ.png)

![](https://i.imgur.com/p6yiU8Q.png)
    
</center>

From Desktop PC ***the attacker accessed several files as well and archived them into loot.zip file at 02:46:18***. It can be seen that there's a symbolical link `loot.lnk` to loot.zip archive.
 
<center>

![](https://i.imgur.com/fT9rPS3.png)
    
![](https://i.imgur.com/pjP97U8.png)

</center>

***The files the attacker was interested in were in /Users/mortysmith/Documents directory***.

<center>

![](https://i.imgur.com/L9c2osE.png)

folder of interest

![](https://i.imgur.com/JspukfF.png)
    
mortysmith's files

</center>

***The attacker created an archive file called `loot.zip` at 02:46*** probably to add files to it. The file was deleted, but traces to it like web cache and symbolical link to it were left. He, most probably, took the archive file out and deleted it afterwards.

<center>

![](https://i.imgur.com/GgNtT80.png)

</center>

It can be seen from web cache that ***the attacker accessed these files from Internet Explorer browser at 2:46***.

<center>

![](https://i.imgur.com/ySIjvi7.png)

Microsoft Edge history
    
![](https://i.imgur.com/bUuIStJ.png)

Microsoft Edge history contents from the .dat file

</center>

### Evil Morty (1st viewpoint)

Hmm, my version of myself from this realm seems to be fond of Jessica. Good boy.

<center>

![](https://i.imgur.com/8D8iHrL.png)

</center>

### Council of Ricks (1st viewpoint)

And lastly, ***he terminates the connection to the Desktop PC by logging off at 02:52:13***.

<center>

![](https://i.imgur.com/ylUgHnH.png)

</center>

***He also logs off from the server right after exiting Desktop PC at 02:52:46***.

<center>

![](https://i.imgur.com/ZOCVigh.png)

</center>

### Evil Morty (1st viewpoint)

Yes, not only did I get my hands on the recipe of the long-cherished Szechuan sauce, but was able to do some tricks to Rick for everything he did to me.

### Narrator

And they lived happily ever after... Or at least I hope so...

---

# Appendices

## Timeline

<center>

![](https://i.imgur.com/anML3fs.png)

</center>

## Additional Figures

The whole process of infection

<center>
    
![](https://i.imgur.com/E7LW9bM.png)
    
</center>

The graph of file modifications

<center>

![](https://i.imgur.com/emiUMO2.png)

</center>

User logins

<center>

![](https://i.imgur.com/Rtbpmo9.png)

</center>

### Evil Morty (1st viewpoint)

Rick, you thought I couldn't get this? You were too carefree...

<center>

![](https://i.imgur.com/aDTuww4.png)

</center>

Now I possess all password hashes of your pathetic family.

<center>

![](https://i.imgur.com/L5hfqKS.png)

</center>

Now I only need to wait until their hashes are removed.

<center>

![](https://i.imgur.com/fc3HGbc.png)

</center>

**10 hours later**

Damn, if only I got enough computational power...

## Analysis tools
* [Autopsy](https://www.autopsy.com/)
* [Volatility3](https://github.com/volatilityfoundation/volatility)
* [Wireshark](https://www.wireshark.org/)
* [Belkasoft Evidence Center x64](https://belkasoft.com/ru/bec/en/Evidence_Center.asp)
* brim 
* snort
* tcpdump
* ClamAV
* [Threat Crowd](https://www.threatcrowd.org/)
* [Hybrid Analysis sandbox](https://www.hybrid-analysis.com/)
* [Virus Total - hash checksum check](https://www.virustotal.com/)