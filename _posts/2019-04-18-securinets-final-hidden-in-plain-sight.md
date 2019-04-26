---
layout: post
current: post
cover: assets/images/securinets.jpg
navigation: True
title: Securinets 2019 Final - Hidden in plain sight forensic challenge
date: 2019-03-24 18:10:00
tags: Writeup CTF Forensic
class: post-template
subclass: "post tag-forensic tag-ctf"
author: Edznux
---

Blackfoot Security went to the Securinets CTF Final event @ Tunis the April 14 2019.

Hidden in plain sight was an easy forensic challenge that gave us **408pts**.

The challenge is a network capture in the standard pcap format named `for2.pcap`.
Openning it in wireshark quickly revealed something interresting: there is a lot of icmp (ping) packets.

When looking a little bit deeper, we saw that most of these icmp packets have a length equal to 60.
The last 2 bytes of each of those are some random looking ASCII character.

![Decoded](/assets/images/hidden-in-plain-sight/icmp.png)

Let's dump all those ICMP packet to plaintext file with tshark (cli version of wireshark)

```bash
tshark -r for2.pcap -T fields -e data "icmp && frame.len == 60" > for2.icmp.txt
```

The file look like :

```
5f260600000000005353
a644060000000000426a
0f570600000000006233
59680600000000005673
07780600000000005a43
c78e060000000000426f
96b60600000000005958
1bcc0600000000005a6c
<snip>......</snip>
eb420b00000000004d6a
505b0b00000000006733
b5a80b0000000000596d
1abc0b00000000005534
47d90b00000000005954
f9ed0b00000000006468
```

At a first glance, only the last 4 digits seem to be in the ASCII range, so we started dumping it.
With some command line tools, we can get the last 4 number and convert them into a single line.

```bash
cat for2.icmp.txt | cut -c17-20 | tr -d '\n'
```

We get the following line : `5353426a623356735a43426f5958...`

Now, because I'm lazy, I used the awesome [CyberChef](https://gchq.github.io/CyberChef/) tool to convert it in readable form.
First from hex to ASCII, then it looked like base64, so I added the "from base64" decoder.

![Decoded](/assets/images/hidden-in-plain-sight/cyberchef.png)

The final flag is : `securinets{1249ce19ef6180e143506b5287be8a7a}`

I started using Scapy, but my setup failed and this was simple enough to be done that way.
Next time, a simple for loop with a filter on ICMP packet should be fine :).