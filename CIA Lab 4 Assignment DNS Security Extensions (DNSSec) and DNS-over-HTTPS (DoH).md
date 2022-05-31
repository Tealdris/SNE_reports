###### tags: `CIA`
:::success
# CIA Lab 4 Assignment: DNS Security Extensions (DNSSec) and DNS-over-HTTPS (DoH)
**Name: Radomskiy Ilia**
:::


## Task 1: DNS Insecurity
:::info
In what context was the DNS cache poisoning attack discovered? (Who, when, why)
:::

Dan Kaminsky finds out that it is possible to poison DNS cache. In 2008 he revealed this DNS weakness to the world. This attack method allows redirecting users from normal websites to imposter websites by changing records in the DNS server[1]

:::info
How does this attack work and how to prevent it?
:::

Request from the server contains in it a 16-bit ID, in order to prove that this is an authoritative server. Dan Kaminsky proves to the world that it isn't that hard to guess this id. Cache poison attack on DNS server can be performed by flooding this server with the malicious IP for a domain with a different ID for each response. If the guess is correct then the DNS server remembers the new malicious IP and, from this point will send every user to this IP. You may see the process of cache poisoning in picture 1[2]

<center>

![](https://i.imgur.com/j6o8U8B.jpg)
Figure 1: Cache poisoning attack
</center>

When the id of the server was simple the were only 65,536 possible ID's, Then it was customary to use id numbers together with the random port numbers in order to make the process of guessing harder.
Some ways of preventing poison attack
You can disable caching, 
you should include only third-party technologies on the server if you really trust them, even better if you keep the DNS server standalone.
Also, it is better if your server has a small amount of trust relationship with the other servers
Of course, you must keep an updated version of the DNS
DNSsec is one of the options in cache poison prevention [3]

:::info
What is DNS spoofing and what tools can be used to integrate this attack?
:::

Spoofing is the attack method where a hacker redirects a user's request from one website to another, cache poisoning is one of the methods
Such tools like ```ettercap``` or ```mitmf``` (man in the middle framework) (Linux)

:::info
What does a validating resolver do? What is the difference between island-of-trust versus full-chain-of-trust?
:::

Validating resolver - is a filter that passes through only information that can be trusted or signed
island-of-trust - it is a zone that has been signed with the key but doest have sinature from parent. Key is configured in a validating recursive name server. It is called an island because it stands alone and doesn't have authentification from parents.
full-chain-of-trust - create a chain of trust means to delegate signing authority, or allow to zone to pass signing authority further, to zones child on the picture 2 you may see the chain of trust. [4],[5]

<center>

![](https://i.imgur.com/GyvsjC4.png)
Figure 2: Validation of the chain of trust from a www.eurid.eu

</center>


:::info
Does DoH substitute DNSSec or complement it? What are the differences between them?
:::

DoH (DNS over HTTPS) - data encryption method, in this case, data is not plain text, and it gives some privacy, it uses encrypted tunnels like in TLS. It uses port 443. It is designed for web applications.
DNSSec (secured version of DNS) - method of validation of information (integrity and authenticity), in this case, data is not encrypted, but you, some kind of, can trust the source of the information (chance of forgery is low)
DoH and DNSSec are facing different problems. DoH - data privacy and encryption. DNSSec - data authentification and validation. That difference makes them a great couple for privacy and confidentiality by fighting different problems and defending from different vulnerabilities

:::info
What do you think about DoH vs DoT vs DNSCrypt?
:::

A DoT (DNS over TLS) - in DoT user are switched from the regular DNS port (53) to 853, and have established a TLS tunnul connection. TLS (transport layer security) is made for security and data confidentiality, it also uses cryptography to this end

DoH (DNS over HTTP) - Method of cryptographic encryption of DNS request. Encrypted tunnels are used in the process, like in TLS, but usage of the HTTPS application layer and different port (443) makes it a little bit different from TLS

DNSCrypt - a server that uses DNSCrypt protocol send to the user set of the verified certificates, the are used as short-term public keys, in order to communicate between user and server, after certain time this set is changing to another one

---
## Task 2: Validating Resolver

:::info
1 Setup
:::

Let's reinstall bind9 because at some point we might need a DoH, that is not supported by the older versions of the bind. The new version of a bind is 9.17.17

***Enable DNSSec to your BIND and verify the root key is used as a trusted source.***

In order to enable DNS, I changed file ```named.conf``` by adding

```
dnssec-enable yes;
dnssec-validation auto;
```

From this point, we might check zones on using DNSSec
You may see example in picture 3

<center>

![](https://i.imgur.com/6tRG0Sg.png)
Figure 3: DNSSec records  on the root server
</center>

***Use dig or drill to verify the validity of DNS records for isc.org, sne21.ru***

Also servers like ```isc.org, sne21.ru``` have their own signatures, you may see them in picture 4

<center>

![](https://i.imgur.com/NAsFfJW.png)
Figure 4: ```dig```ing in the ```isc.org, sne21.ru```
</center>

:::info
2 Validate
:::

***How does dig or drill show whether DNSSec full-chain-of-trust validation was successful or not?  Hint: it’s about a flag***

Information from the ```dig``` output might tell us some information about the server

AD (Authentic Data): indicates the resolver believes the responses to be authentic - that is, validated by DNSSEC. And it on the ```sne.ru``` as well
RD (recursion desired)
RA (recursion available)

:::info
2 Counter-validate (breaking the chain of trust)
:::

***Where does BIND store the DNSSec root key?***

Root keys a loaded in the ```bind.keys``` file [6]

<center>

![](https://i.imgur.com/XnTu6CA.png)
Figure 5: Storadge of the root kay
</center>

***How do managed keys differ from trusted keys? Which RFC describes the mechanisms for managed keys?***

If the local resolver trusts some key, then those keys are stored in the trusted keys.

In the case of managed keys, they are stored in the ```managed-keys```, and this file will updates if the option ```dnssec-validation auto; ``` is enabled

<center>

![](https://i.imgur.com/Mu7zfRG.png)
Figure 6: ```managed-keys``` storadge
</center>



***Modify the DNSSec root key and test your validating resolver.***

The file that contains the root key tells, that the user shouldn't change any record, but in case of experiment we might have a try, you can see the result on picture 7

<center>

![](https://i.imgur.com/83HkmlM.png)
Figure 7: Error occurred because of changes in the root 
</center>

***How did your server react?***

As you can see the loading of the bind failed because of the changed key.


## Task 3: Secure Zone

:::info
Island of trust
:::

***Look up which cryptographic algorithms are available for use in DNSSec. Which one do you prefer, and why?***

DNSsec supports several cryptographic algorithms, some of them, because of their insecurity and "weakness" are not longer widely used, or not recomended[8]

<center>

![](https://i.imgur.com/fofWHJD.png)
Fiugre 8: cryptographic algorithms is used in DNSSec 
</center>

RSA/SHA-256 is my choice, it is popular, a lot of programmes supports it, it can be used in both signing and validation.

***What is the difference between Key-Signing Key (KSK) and Zone-Signing Key (ZSK)?***

KSK (key signing key) - it is a key that is used to verification between zones and for key verification, from parent and the child. It is changed rarely, in order to have a more stable entrance point into the zone
ZSK (zone signing key) - it is an authenticating key that verifies the zone itself (every record in zone), this key is verified by KSK. It changed more often than KSK.

***Why are those separated and why do those use different algorithms, key sizes and lifetimes***

KSK is longer than ZSK and it changed less often, only signs DNSKEY RRset, another to validate inside the zone or signs all RRset-s in the zone.
ZSK Can be lower strength than the KSK, because this key is not sent to a parent or child zone, if it is modified, only local changes happen. That is why it is shorter - because it is easier to change it in case of compromise 
KSK on the other  hand need to be specified in the parent zone, and because of the possible huge amount of children of one parent and  lot of work  and expenses, KSK  changed rarely, and is stronger
Size of the key depends on the best-complitted attack on the key, according to RFC 6781 [9], this number is 700 bit, which means that 1024 asymmetric key is secure enough.

***Use BIND9 tools (dnssec−keygen) to secure your zone. Show the configuration files involved and validate your setup***

First step we need to take in order to zone securing is key creation and their signing in into zone config file [10]

<center>

![](https://i.imgur.com/tC8sVAL.png)
Figure 9: key generation
</center>

Then I changed my zone file, by adding public keys to the file

<center>

![](https://i.imgur.com/D8I59Tx.png)
Figure 10: ```db.st10.sne21.ru``` file
</center>

After that, I can sign the zone with the command

```
dnssec-signzone -e +3mo -N INCREMENT -K /usr/local/etc/bind/keys/ -o st10.sne21.ru -t ../db.st10.sne21.ru
```

in the command I specified:
+3mo - time of signing - 3 month
-N INCREMENT - do not modify SOA record
-K - keys folder
-o - name of the zone
-t - and location of the zone

After command implementation our zone file is ```signed``` and we need to put it in the ```named.conf``` file

<center>

![](https://i.imgur.com/wLY47gM.png)
Figure 11: Updated ```named.conf``` file
</center>

From this point, our server have a DNSSec record 

<center>

![](https://i.imgur.com/8d6CLss.png)
Figure 12: Recorsd of the server via ```dig```
</center>

On the picture we cant see ```AD``` flag thatt means our server not sending Authentic Data yet

:::info
Full chain of trust
:::

On this point, I have created an amazon cloud machine, in order to use its public IP, so I can create ```st10``` zone in the ```sne21``` zone

***Does your parent zone offer secure delegation?***

Yes, and by writing several records in the ```sne21``` zone, we can validate our own zone

<center>

![](https://i.imgur.com/5Wk4G2p.png)
FIgure 13: getting fully validated zone output
</center>

***Describe the DS and DNSKEY records from sne21.ru that are important for your domain. Which keys are used to sign them?***

We might see several records, as a response of our server:
DS - hash of the KSK, this record signing keys in the zone, by the record itself we can find that server is using RSASHA256 algorithm

RRSIG - a signature of the NS records and DS records, that proves theirs reliability

<center>

![](https://i.imgur.com/gNHi5h7.png)
FIgure 14: dnssec records in the ```sne21``` zone
</center>

We also can use web site ```dnsviz.net``` in order to check how does my zone look like, and does procedure of the dns verification 
<center>

![](https://i.imgur.com/A3KHWVq.png)
Figure 15: DNS check
</center>

---
## Task 4: Key Rolloveres

:::info
Zone-Signing Key rollover
:::

***Study RFC 6781 or affiliated online resources***

***What are the options for doing a ZSK rollover? Choose one procedure and motivate your choice***

First - key pre-publication uses if the key is compromised, and the new one is already in uploaded
+Size of the zone does not double,
don't sign the zone twice;
-This creates extra work for the KSK signing process,
because the key is published it is available to analysis to the third-party,
this process is longer than the next one

Second - double signatures in this case new file of the DNS zone is created, then the new ZOne key is advertised t the authoritative servers
+This can be performed in the less amount os stages
-if your zone is very big, that you have to deal with the double sized zone

Because my zone is very small, it is better to be using "Double-Signature ZSK rollover" [9]


***How would you integrate this procedure with the tools for signing your zone? Which timers are important?***

If in DNS time is relative (every server counts time individually), that DNSSec uses absolute time (servers depended on the  time of the exploration of the signature time ), this means that timers on the zones or signatures after the expiration date is not valid
Server owner also have to wait certain time in order to others users to have tome to update their keys

***Bonus (1 point): integrate the procedure and use a DNSSec debugger to verify each
step***

:::info
Key-Signing Key rollover
:::

***Can you use the same procedure for a KSK rollover?***

Yes same considerations are applied as for Key Signing Key Rollovers and for Zone Signing Key Rollovers, but they are different because  the KSK rollovers require interaction with the parent's zone
For example, for the double-sign zone transferring, in the case of the ZSK, we have to create two keys, then, at some point we start using a new key, but does not delete the old one, because someone might still use it
In the case of the KSK, we will be using both keys because of waiting for the parent one to apply new ds record (zone transfer), and then, because of TTL os DS record set in the parent

***What does this depend on?***

The sentences above means that you cant do this without interaction with the parent's zone. When a parent is notified he publish keys "DNSKEY_K_1" and "DNSKEY_K_2", at this moment child needs to replace "DNSKEY_K_1" with "DNSKEY_K_2"

---

## Task 5 - DNS-over-HTTPS

:::info
Implementing DNS-over-HTTPS on your server.
:::

***For this step you probably will need to recompile and reinstall your server (BIND requires Development Release).***

DoH is a relatively new technology, so some of the DNS server program's does support it. In the case of a bind, it is available in Bind.9.17.17 version. It is in the development stage, but we still might use it. To this end, I reinstalled bind, the same way as I did it in the previous lab.

***Also, you will need a certificate for your domain, you can use ```certbot``` to obtain it.***

At this point, we might gain the certificate that allows us to use DoH, in order to do that i performed 
```
sudo certbot certonly --standalone --preferred-challenges http --agree-tos --email r.ilia@innopolis.university -d ns.st10.sne21.ru
```
<center>

![](https://i.imgur.com/c8QIsUn.png)
Figure 15: Getting certificate
</center>

***Show configuration options that have to be tuned.***

Some of the configuration files do require modification. 
cerbot, that was installed also need some configuration files, like ```usr.sbin.named```. This file is used in order to give access to the ```named``` configuration file, because, main config file may be overwritten.

<center>

![](https://i.imgur.com/zNeVdtA.png)
FIgure 16: ```usr.sbin.named``` file
</center>

Then I edit ```named.conf``` file.
There are only 5 lines appended
The FIrst part allows usage of the TLS tunnels in the zone.
```
tls local-tls {
   key-file "/etc/letsencrypt/live/ns1.talkdns.net/privkey.pem";
   cert-file "/etc/letsencrypt/live/ns1.talkdns.net/fullchain.pem";
};
```
And the second stands for the listening on HTTPS port
```
listen-on port 443 tls local-tls http default {any;};
```

<center>

![](https://i.imgur.com/76kPDa5.png)
Figure 17: ```nammed.conf``` configuration file
</center>



***Verify that DNS-over-HTTPS is working (Chrome and Firefox has support for it) by inspecting server incoming queries in the log.***



From this point we might use DOH, and, so by digging to the 
```
dig +https @ns.st10.sne21.ru isc.org A
```
We are getting replies listed below
```
;; SERVER: 51.210.161.197#443(ns.st10.sne21.ru) (HTTPS)
```

[11]

:::info
***Bonus - Secure Zone Delegation (2 points)***
:::

***Securely delegate a subzone to another student - ```st<X>.st<Y>.sne21.ru```, where X - the number of your machine, Y - the number of your partner’s machine.***

***How to communicate the necessary record to your buddy? (Integrity matters here)***

***What about Confidentiality, does it matter here?***


:::info
***Bonus - NSEC vs NSEC3 (1 point)***
:::

***What are the main differences between NSEC and NSEC3?***

Both NSEC and NSEC3 are used in order to prove that records non exist, they provide “authenticated denial of existence”.
The main difference between them is ```zone walking```
So ```zone walking``` is when you don't know what are you exactly looking for inside of the domain, and just gather information about all records in the zone
In the case of the NSEC, it is was unexpected, so it is possible to zone walk, just by brute force
WIth the NSEC3 it is harder to achieve because of the hashing algorithm and salt using

***How to select one or the other during zone-signing?***

We can specify our choice by adding potion -3 and specifying the algorithm in the ```dnssec-keygen``` command

<center>

![](https://i.imgur.com/d9HqvMH.png)
Figure 18: NSEC and NSEC3 in the ```dnssec-keygen```
</center>

[12]

***Do the root-servers use NSEC or NSEC3? What about the .org domain?***

A root server is using an NSEC, you may see prove on picture 19

<center>

![](https://i.imgur.com/klK7P2C.png)

FIgure 19: NSEC in root
</center>

On the other hand, org is using NSEC3

<center>

![](https://i.imgur.com/1dxhAR3.png)

FIgure 20: NSEC3 in org
</center>

***How to take advantage of zone traversal as an attacker, how does it work?***

For example, you may use a website like 
```
https://www.dnsqueries.com/en/dns_traversal.php
```
or utilities like ```ldns-walk```
that will help to resolve some names into IP records in the DNS zone.
The more record in the zone the more vulnerabilities can be found because it is very hard to maintain big systems

---
## References:

1: [DNS cache poisoning](https://arstechnica.com/information-technology/2020/11/researchers-find-way-to-revive-kaminskys-2008-dns-cache-poisoning-attack/)
2: [The Kaminsky Vulnerability: DNS Under Attack](https://www.youtube.com/watch?v=qftKfFVHVuY)
3: [How to protect your infrastructure from DNS cache poisoning](https://www.networkworld.com/article/3298160/how-to-protect-your-infrastructure-from-dns-cache-poisoning.html)
4: [DNSSEC HOWTO](https://www.dns-school.org/Documentation/dnssec_howto.pdf)
5: [DNS SECurit y E x tensions technic al over view](https://eurid.eu/media/filer_public/33/7d/337dfd35-e77f-428a-b951-1668ef719f01/insights_dnssec2.pdf)
6: [datatracker](https://datatracker.ietf.org/doc/html/rfc6781#section-3.4.20)
7: [A Root Key Trust Anchor Sentinel for DNSSEC](https://www.rfc-editor.org/rfc/rfc8509.html)
8: [Domain Name System Security Extensions](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions#Resource_records)
9: [DNSSEC Operational Practices, Version 2](https://datatracker.ietf.org/doc/html/rfc6781)
10: [DNSSec on bind](https://www.dmosk.ru/miniinstruktions.php?mini=dnssec-bind)
11: [DNS Over HTTPS With BIND 9.17](https://www.isc.org/blogs/doh-talkdns/)
12: [dnssec-keygen(8) - Linux man page](https://linux.die.net/man/8/dnssec-keygen)
13: [Traversal Zones - Expressway Series](https://www.youtube.com/watch?v=aCpvWnWiuUk)