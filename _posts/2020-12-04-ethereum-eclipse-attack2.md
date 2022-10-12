---
title: "Eclipse Attack in Geth v1.8.x"
categories:
  - Blockchain
  - Ethereum
date: 2020-12-04 15:00:25 +0900
tags:
  - Network
  - Ethereum
---

# Countermeasures of Eclipse Attack in Geth v1.8.x
* In Geth v1.6.6, If Attacker send Ping to the victim, attacker can monopolize connections of the victim
* To prevent mopolized connection, Geth 1.8.x limit their inbound connection up to 17
* Also in Geth v1.8.x, they limit IP in their DHT.
  * In DHT's bucket, node with same IP can be exist up to 2
  * In DHT, node with same IP can be exist up to 10
  
![img](../../assets/p/4/monoply2.png){: width="300" height="300"}


# But still, Eclipse Attack exists ...
![img](../../assets/p/5/1.png){: width="300" height="300" }

As show in previously, **now the victim use outbound connections for safety connection to normal nodes**. However, outbound connection is also monopolized by the attacker.

* Outbound connection is consist of 2 types of functions(`lookupBuffer`, `ReadRandomNodes`).
  * lookupBuffer is a storage of closest nodes. This is filled when the victim send `FindNode` packets to other nodes.
  * ReadRandomNodes is a function that find nodes for outbound connection from the DHT.

## Monopolize `lookupBuffer`
1. Attacker generate multiple nodes for attack preparation.
2. When the attacker get `FindNode` packets from the victim with random target, using prepared ndoes, attacker send closest nodes to the victim.
3. `lookupBuffer` is filled with attacker's nodes.

## Monopolize `ReadRandomNodes`
`ReadRandomNodes` load the nodes from DHT. **But, only load top of the buckets!**(This is a critical vulnerability that allows the attacker to fill the small portion of buckets)
1. Attacker send `Ping` to the victim
2. victim send `Pong` and fill a attakcer node to top of the bucket.


> Since the victim has only 17 buckets, attacker send 17 `Ping`s to the victim(to fill each buckets). Thus, this vulnerability make the attacker to reduce their resource for the attack.
{: .prompt-warning}


# Analysis of Geth v1.9.24 Network
![img](../../assets/p/5/3.png)