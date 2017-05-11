---
layout: default
title:  "Pfsense dropping packets"
date:   2017-05-11 10:00:00 +0000
categories: pfsense networking
---

Debugging why the uploading of files into some nodes on our network turned out to have a surprising outcome.

###Symptom

In our case, we have an office with pfsense between us and our WAN connectivity. This pfsense box shares the same subnet as our LAN workstations. Additionally, we have another set of computers (test servers) on their own private subnet linked to the main LAN via a second pfsense machine.

We could log in to these test servers and download things just fine. But uploading more than a very small file from our workstations to these test servers results in a connection drop.

###Cause

The office pfsense box realises that the workstation can directly route traffic to the test servers pfsense box. Sadly, Windows does not recognise the signalling involved, and thus the network connectivity is broken.

###Solution

Send a classless static route via DHCP teaching each workstation that the test servers subnet can be reached through the test server's own pfsense box.
