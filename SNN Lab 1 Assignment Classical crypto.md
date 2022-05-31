###### tags: `SSN`
:::success
# SNN Lab 1 Assignment: Classical crypto
**Name: Radomskiy Ilia**
:::

---
## Task 1: Open   the   Codebook   and   look   at   everything   up   to   and   including   Vigenere cyphers:  choose  “Main  Contents”  and  go  through  the  first  three  chapters  of  the  “Birth of cryptography” up to and including “Mechanising secrecy”

Firstly, download Win XP with the installed codebook in it, then open it and read through

 ![](https://i.imgur.com/nGubYSC.png)

---
## Task 2: Encrypt an   English text of at least   80   words using the   Vigenere cypher and exchange it with one of your fellow students. 

<center>

![](https://i.imgur.com/I1Gn3Jb.png)
Vigenere table
</center>

My text:

```The first book of the Satanic Bible is not an attempt to blaspheme as much as it is a statement 
of what might be termed "diabolical indignation". The Devil has been attacked by the men of 
God relentlessly and without reservation. Never has there been an opportunity, short of 
fiction, for the Dark Prince to speak out in the same manner as the spokesmen of the Lord of 
the Righteous. The pulpit- pounders of the past have been free to define "good" and "evil" as 
they see fit, and have gladly smashed into oblivion any who disagree with their lies - both 
verbally and, at times, physically. Their talk of "charity", when applied to His Infernal 
Majesty, becomes an empty sham - and most unfairly, too, considering the obvious fact that 
without their Satanic foe their very religions would collapse. How sad, that the allegorical 
personage most responsible for the success of spiritual religions is shown the least amount of 
charity and the most consistent abuse - and by those who most unctuously preach the rules of 
fair play! For all the centuries of shouting- down the Devil has received, he has never shouted 
back at his detractors. He has remained the gentleman at all times, while those he supports 
rant and rave. He has shown himself to be a model of deportment, but now he feels it is time 
to shout back. He has decided it is finally time to receive his due. Now the ponderous rule- 
books of hypocrisy are no longer needed. In order to relearn the Law of the Jungle, a small, 
slim diatribe will do. Each verse is an inferno. Each word is a tongue of fire. The flames of 
Hell bum fierce . . . and purify! Read on and learn the Law. 
```

I will be using the word `DEAD` as my keyword. By looking at the Vigenere table we can code the word `the` like `wle`. So, going through the text I can get fully ciphered text, like in the text below.

```
Wle ilvsw eson rj tkh Wawdrif Emboh ms qrx aq dxthptt wr fldvthhpi av pyck dw iw lw a vwethpinw
rj wkdx mljlt eh xeupid "glebromcdo mnglkndwmoq". Wle Ghzio kes ehin dwxafnid eb xhh pin ri
Kog uilhqxlhvwlb drd zlxhrxx rhvirydxirq. Reyhv hdv xhhui bhhr aq rtpruxuqlxy, vksrw rj
flfxirq, jou wle Gdvk Sumnfh xo vsian ryt lq xhh vemh penqhv av wle vsskhvqeq rj tkh Poug sf
wki Rljlthrys. Wki pxotiw- ssuqgirv rj tkh tavw layh fehq jrhh xo ghjiqh "korg" eng "hzio" dw
tkhc shh jiw, drd kdze joedob wmdvleg lrtr rfllymoq dry zks dlveguhi wlwl tkhmr olis - erxh
yhvbdopy dqh, aw wmmhv, thbvmcdopy. Wkiiu weln rj "ckdviwb", ahhq epsomeg ws Hlv Mnihvndo
Qamhwtb, eicrpis dq imswc skdq - aqg qovw ynidmrob, xor, fsnvlheulrg wki oeymoxv jafw xhdw
aiwksuw wlelu Wawdrif ise wkiiu yirb uilljmoqv aoxoh cropasvi. Hrz wag, wlaw wle dopejrvifdp
phuwoqdke prwt uhwprqwieoi fru xhh vycfhws ri wplumtxdp rhomglrrs lv whrzr tkh pedvx aprynw rj
ckdviwb eng wle prwt frrslvxeqw ebxvi - aqg fy wkssh zlo prwt xqgtxrysob trhdgh wki rxois ri
jalu tldb! Jou dpl wki chqxuulis ri whrxxiqj- hozq xhh Givlo lav uichlzeg, ki hdv reyhv skrythg
fafn et klw dhwvafwsrv. Ki hdv vepdmnhg xhh jinwoimdq et dop tlpis, zkmlh wlovh le vxtpruxs
udrt dqh rdyi. Hh kes vkswq kmmvhpf wr fe d psdho sf ghtouwqeqw, fuw qsw kh jehow iw lw tlpi
tr vloxw fafn. Le kdw dhfmdhg mt lv jiqdplb wmmh ws rhfiiyh liv gye. Qra tkh toqgirrxw rxoi-
brros ri lysrgrlvc auh ro orrghu rehgid. Lq srghv tr uilhdvn wki Ldz sf wki Jxqklh, d wmdop,
solq dldxrlei wlop dr. Heck yirvh ms dq mnihvnr. Heck zsrg lw a wrrgxh sf ilve. Wki fodqev rj
Hhop bxp jihuge . . . dqh pxumfb! Uiag rr aqg pedur tkh Paz. 
```

And then send to the Vladimir.

---
## Task 3: Crack the encrypted text of your fellow student using the Vigenere cypher tool. 

The coded message that was sent to me by Vladimir

```
Grnp sdie s acofmrsr rlvsfs, hzmwp G dievslpv, apli ohu osucq, Sgpp aueq o kfsmye ybx tmfczmw gzjigv gt zzjkzersh cgfy, Hzmwp G biuvsx, ywecww bughwhr, kyoocbfp lvycw glxc o nrhdcyy, Ed zd gidw chp yiyejm lrhdcyy, vlanwhx sh gj ullxzsl ugcl. Eaw dzks pzkwnpj, M xfrhyiwr, nlhttye on dq qblefpc bcii-Gbfj lltd, ybx eghbtfk xzps. Uy, vwmearnejm C iwayxtic tr kuj ab nsw fwpyy Xvusgmwv, Lyb sutz gyasvlec rszfu yxtic hpcoxzh cek kszqh oggb nsw jwzmf. Yryslwq M htqvyu lvy xgvczu;-juzfzs T zeo dmiayl hi mgvczu Tlfe as mgsvd qiltwomp gj dzpfin-kclcga qzp hbv dcme Diyzps-Zfj hbp jecp ybx isrclfx xlgrye ovix llp lluyck buxw Ppymfy-Esaywwwd scfy wgf ygwvxzps.
```

In order to decode this message below steps were taken:

1. Guessing key length
For this step, I have written python code that analyses text, as a result of the program I get 3 values (triplet, his index, and amount of repetitions)

<center>

![](https://i.imgur.com/xMaCzJ7.png)
output of python code
</center>

By finding the same triplets and distance between them we can geuss that key length is one from `12 8 6 4 3 2`


| triplet | distant | devisors |
| -------- | -------- | -------- |
| zmw     | 96-24=72| 24 12 8 6 4 3 2     |
| dcy     | 165-141=24     | 12 8 6 4 3 2    |
| tic     | 324-276=48     | 24 12 8 6 4 3 2     |
| etc     | ...     | ...     |


3. Guess the key itself

Then we need to divide the text into several cryptograms, number same as key length
After that, we can implement frequency analysis (amount of repetitions of letters in English), and compare it to the cryptograms
If every step that has been taken before is correct, then our key will be correct. In the case of Vladimir, he coded his message with the key `sellyoursoul` word.
At this step, we can try to decode the whole message

The decoded message, key `sellyoursoul`

```
ONCE UPON A MIDNIGHT DREARY WHILE I PONDERED WEAK AND WEARY
OVER MANY A QUAINT AND CURIOUS VOLUME OF FORGOTTEN LORE 
WHILE I NODDED NEARLY NAPPING SUDDENLY THERE CAME A TAPPING 
AS OF SOME ONE GENTLY RAPPING RAPPING AT MY CHAMBER DOOR 
TIS SOME VISITER I MUTTERED TAPPING AT MY CHAMBER DOOR 
ONLY THIS AND NOTHING MORE 

AH DISTINCTLY I REMEMBER IT WAS IN THE BLEAK DECEMBER 
AND EACH SEPARATE DYING MBER WROUGHT ITS GHOST UPON THE FLOOR 
EAGERLY I WISHED THE MORROW VAINLY I HAD SOUGHT TO BORROW 
FROM MY BOOKS SURCEASE OF SORROW SORROW FOR THE LOST LENORE 
FOR THE RARE AND RADIANT MAIDEN WHOM THE ANGELS NAME LENOREN
```

We also can use some internet resources, like [dcode](https://www.dcode.fr/) to crack codes

<center>

![](https://i.imgur.com/abyFJiu.png)
dcode site
</center>

or can use codebook to this end
<center>

![](https://i.imgur.com/E0qKBPT.png)
Code book vigenere cracking tool
</center>

[How to crack Vigenere](https://www.quora.com/How-can-I-crack-the-Vigenere-cipher-without-knowing-the-key)

---
## Task 4: Go through the previous two steps again,  this time using a cypher of your own choosing. Do not tell your fellow student what cypher you used 

I will be using the Atbash coding method for this part of the task. 

<center>

![](https://i.imgur.com/wcGgvas.png)
</center>

In terms of this task, I will cypher the text listed below

```
The Polybius square, also known as the Polybius checkerboard, is a device invented by the ancient Greeks Cleoxenus and Democleitus, and made famous by the historian and scholar Polybius. The device is used for fractionating plaintext characters so that they can be represented by a smaller set of symbols, which is useful for telegraphy, steganography, and cryptography. The device was originally used for fire signalling, allowing for the coded transmission of any message, not just a finite amount of predetermined options as was the convention before.
```

Fully ciphered sentences is listed below
```
Gsv Klobyrfh hjfziv, zohl pmldm zh gsv Klobyrfh xsvxpviylziw, rh z wverxv rmevmgvw yb gsv zmxrvmg Tivvph Xovlcvmfh zmw Wvnlxovrgfh, zmw nzwv uznlfh yb gsv srhglirzm zmw hxslozi Klobyrfh. Gsv wverxv rh fhvw uli uizxgrlmzgrmt kozrmgvcg xszizxgvih hl gszg gsvb xzm yv ivkivhvmgvw yb z hnzoovi hvg lu hbnyloh, dsrxs rh fhvufo uli gvovtizksb, hgvtzmltizksb, zmw xibkgltizksb. Gsv wverxv dzh lirtrmzoob fhvw uli uriv hrtmzoormt, zooldrmt uli gsv xlwvw gizmhnrhhrlm lu zmb nvhhztv, mlg qfhg z urmrgv znlfmg lu kivwvgvinrmvw lkgrlmh zh dzh gsv xlmevmgrlm yvuliv.
```

Also in this task, I have to decode the message of one of the students, Alisher, that he has sent me.

```
RKPBVZKYGQLWBXPHUFKLOGROGQDQVDLGTXLHWOBZKDWGRBRXNQRZRIIHDUIHDULVIRUWKHZLQWHUPBOLCZNXEOSDKYIDWTKFQXEYJDHDBVDKXQGUHGIHHWGHHSDQGWKHLFHZLQGFRPHVKRZOLQJRXWRIWKHQRUWKIHDULVIRUWKHORQJQLJKWZKHQWKHVXQKLGHVLWVIDFHIRUBHDUVDWDWLPHDQGOLWWOHFKLOGUHQDUHERUQDQGOLYHDQGGLHDBVUKTLEXUNQHAZAZKZLOHWKHGLUHZROYHVJURZJDXQWDQGKXQJUBDQGWKHZKLWHZDONHUVPRYHWKURXJKWKHZRRGVA
```

The first thing to use to crack this message is to use a frequency analyzer

Because frequencies are not averaged we can make an assumption that it is one of the substitutions cyphers

![](https://i.imgur.com/UwusdzO.png)

by looking at the picture we can make a guess that letter `h` represents the letter `e` as the most frequent letter in the English alphabet

![](https://i.imgur.com/rxcvU6n.png)

by looking at one of the codebook tools we can continue our assumption, in this time we adding to it: `k` is `h` in the English alphabet

In our next step, we are getting from the `letter pairs help tool`, we already make one assumption about the alphabet, so now we trying a new one: in this case, as in the previous one we are making guesses on the most frequent pairs, `eh` in the first case, and `th` in the second case

![](https://i.imgur.com/gbhDkCE.png)

At this stage we might find `the` in the message, and, as you can see in the picture below we already have some results

![](https://i.imgur.com/eDP8MSn.png)

The next tool to use is `vowel trower`, which can give us a hint about vowels.
After `e` the most frequent vowels is `a` and `o` so we are guessing that `r`-`o`, `d`-`a`. Or vice versa

![](https://i.imgur.com/9OSC9jo.png)

increase our confidence in assumption may give as `frequency analysis of repeated letters`, according to this tool there more chance to get double `o` than `a`, so we thinking that `r` equals to `o`

![](https://i.imgur.com/aJjUmxf.png)

another hint is the only pair in the message of paired o's

![](https://i.imgur.com/Dk0Rp0C.png)

We can make guesses and further, but we might notice the picture below

![](https://i.imgur.com/oGlhhOP.png)

Like we are having a pretty good chance of having some of the offset cyphers
So we started working on this guess
By filling all of the empty spaces, we are getting decoded message

![](https://i.imgur.com/TaUnY3U.png)

by googling text in it we have found that we are successfully decoded message

![](https://i.imgur.com/g5Lcxhz.png)

After decoding we can guess that Alisher was using Caeser-3 for this message

---
## References:

[How to crack Vigenere](https://www.quora.com/How-can-I-crack-the-Vigenere-cipher-without-knowing-the-key)

