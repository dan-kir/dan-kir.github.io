---
layout: post
title:  "Experiences running a Tor exit relay for almost 2 years"
date:   '2022-06-01'
author: Dan Kir
tags:   
  - privacy
  - tor
  - linux
description: >-
  Experiences running a Tor exit relay for almost 2 years
---

I have been running a Tor exit relay with 1984 hosting (https://1984.hosting/) for almost 2 years.

Below are some notes and observations on experiences so far.

If you can identify my relay and DM me the fingerprint I'll buy you a beer and credit you in this post.

|![Alt text](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/tor-relay-history-1.png "Tor exit relay history")|
|:--:|
|<b> Relay bandwidth history </b>|

|![Alt text](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/tor-relay-history-2.png "Tor exit relay probability history")|
|:--:|
|<b> Relay probablity history </b>|

#### Configuration management

There are many Ansible roles available. The most common choice is ansible-relayor (https://github.com/nusenu/ansible-relayor)

I chose to create and maintain my own instead (https://github.com/dan-kir/ansible-debian-11-tor-relay). This role is much simpler, however only supports Debain based operating systems.


#### Abuse notices

During the lifetime of this relay, I have received two abuse notices.

On both occasions I was missing a reverse DNS pointer record (PTR) that would have made it more obvious that it's a Tor exit relay (https://community.torproject.org/relay/setup/exit).

The first notice was from CERT-IS. The relay was likely involved in the distribution or C2 for the Emotet malware. I wasn't quick enough to respond and 1984 hosting suspended the VPS temporarily.

After an email to customer support, an updated PTR record and a quick security assessment the VPS was fully restored.

| ![Abuse notice #1](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/1984-abuse-notice-1.png "Abuse notice #1") |
|:--:|
|<b> Abuse notice #1 - Emotet infection</b>|

The second notice was from SK Telecom. Seemed the relay was routing traffic for someone trying to access something they shouldn't. This is why we can't have nice things.

I had recently reprovisioned the VPS and forgot to update the PTR record.

| ![Abuse notice #2](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/1984-abuse-notice-2.png "Abuse notice #2") |
|:--:|
|<b> Abuse notice #2 - HTTP intrusion attempt </b>|

#### Monitoring with Nyx

Nyx (https://nyx.torproject.org/) is a command-line monitor that shows real-time stats. Can be useful for troubleshooting. It is also just nice to watch sometimes.

| ![Nyx monitoring](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/tor-relay-nyx.png "Nyx monitoring") |
|:--:|
|<b> Nyx monitoring </b>|


#### Tor swag

If you run a fast relay, or have contributed to the project in some other way, you might be eligible for a Tor T-Shirt (https://community.torproject.org/relay/community-resources/swag/)

I am still waiting for mine.

#### Summary

Operating a relay has been a bit of fun. It doesn't cost much and there is no significant admin overhead. It is set and forget, for the most part.

If you have used Tor in the past, you should consider running a relay yourself. Operating a relay from your home or work place might not be a smart idea, so I would recommend checking out 1984 hosting.
