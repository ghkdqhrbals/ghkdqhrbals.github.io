---
title: "Eclipse Attack in Geth v1.6.6"
categories:
  - Blockchain
  - Ethereum
date: 2020-11-27 15:00:25 +0900
tags:
  - Network
  - Ethereum
---

# Analysis of Geth 1.6.6v's P2P network
![img](../../assets/p/4/geth166.png)
![img](../../assets/p/4/1.png)
* create UDP/TCP listener
* schedule 2 types of task(discoverTask, dialTask)

![img](../../assets/p/4/2.png)
* seeding Ethereum nodes into Kademlia DHT

![img](../../assets/p/4/3.png)
* create Kademlia DHT

![img](../../assets/p/4/4.png)
* get Ethereum node's information from local database(level db) for seeding

![img](../../assets/p/4/5.png)
* get `seedCount`(default = 30) Ethereum nodes from DB and also hard-coded bootstrap_nodes.
* insert `bootstrap_nodes` + `DB_nodes` into `DHT`

![img](../../assets/p/4/6.png)
* keep DHT fresh and do bonding process
> red-box is a goroutine

![img](../../assets/p/4/7.png)
* use lookup(random target) process to populate DHT
* validating nodes in DHT with Ping/Pong pakcets(1h)
* keep the old nodes in DHT in DB(5m)


![img](../../assets/p/4/8.png)
* connection read loop

![img](../../assets/p/4/9.png)
* create TCP Listener

![img](../../assets/p/4/10.png)
* create 50 channel for connection and scheduling

![img](../../assets/p/4/11.png)
* pass the results of handshake to srv.run goroutine's channel
* add peer to eth.peer

![img](../../assets/p/4/12.png)
* RLPx handshake
* set both in/outbound connection


![img](../../assets/p/4/13.png)
* goroutine
* 2 types Task scheduling(discover, dial)
* By its channel, add peer and execute

![img](../../assets/p/4/14.png)
* discoverTask : kademlia-like lookup
* add nodes into lookupBuf when discoverTask is done
* dialTask : dialing for setupConnection

![img](../../assets/p/4/15.png)
* kademlia-like lookup

![img](../../assets/p/4/16.png)
* Get closest(random Target) 16 nodes from DHT
* Among these nodes, if there are first-seen-nodes, doing bonding process and insert into db.

![img](../../assets/p/4/17.png)
* run when running task is below 16
* MaxDynamicDial(MaxOutboundConn) = ( 1+maxpeer(25) )/2 = 13
* can add staticNode
* leftover dialing count = `needDynDials`
* extract 6 nodes( `needDynDials`/2 ) from DHT, and create dialTask
* if it still require more nodes, extract 7 nodes from `lookupBuf`, and create dialTask

![img](../../assets/p/4/18.png)
* Where Inbound connection are actually confirmed
* Inbound connection is no limited

> Attacker create a lot of inbound connection requests to the victim in 20 seconds right after the victim boot.
{: .prompt-warning}

Thus, to prevent all connection from dominated by adversary connections, in Geth 1.9.24v, they set limit of inbound connections. So if attacker send multiple inbound connection requests, there is a limit! And the victim can still connect with normal nodes by outbound connection.
![img](../../assets/p/4/19.png)
