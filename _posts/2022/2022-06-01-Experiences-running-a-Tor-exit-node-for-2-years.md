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

I have been running a Tor exit relay with 1984 hosting ([https://1984.hosting](https://1984.hosting)) for almost 2 years.

Below are some notes on experiences so far.

If you can identify my relay and DM me the fingerprint I'll buy you a beer and credit you in this post.

|![Alt text](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/tor-relay-history-1.png "Tor exit relay history")|
|:--:|
|Relay bandwidth history|


|![Alt text](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/tor-relay-history-2.png "Tor exit relay probability history")|
|:--:|
|Relay probablity history|



#### Configuration management

There are many Ansible roles available. A common choice is ansible-relayor ([https://github.com/nusenu/ansible-relayor](https://github.com/nusenu/ansible-relayor))

I chose to create and maintain my own instead ([https://github.com/dan-kir/ansible-debian-11-tor-relay](https://github.com/dan-kir/ansible-debian-11-tor-relay)). This role is much simpler, however only supports Debain based operating systems.

The configurations in defaults/main.yml are pretty standard. Includes ports, badwidth rates and the exit policy.

```yaml
---
tor_distribution_release: "{{ ansible_lsb.codename }}"

tor_orport: 9001
tor_dirport: 80
tor_controlport: 9051

tor_firewalld_config_enabled: yes

## This user is created during package install
tor_user: debian-tor

## Will relay is exit is not enabled
tor_exit_enabled: yes
tor_exit_notice_enabled: yes

tor_nickname: "{{ machine_hostname }}"

tor_contact_info: admin@email.com

tor_bandwidth_rate: 1.5 #MBytes
tor_bandwidth_burst: 3 #MBytes
tor_accounting_max: 100 #GBytes

tor_exit_notice_file: tor-exit-notice.html

tor_data_dir: /var/lib/tor
tor_config_dir: /etc/tor/

## Only relevent when exit enabled
tor_exit_policy:
  - accept *:20-21     # FTP
  - accept *:43        # WHOIS
  - accept *:53        # DNS
  - accept *:80        # HTTP
  - accept *:110       # POP3
  - accept *:143       # IMAP
  - accept *:220       # IMAP3
  - accept *:443       # HTTPS
  - accept *:873       # rsync
  - accept *:989-990   # FTPS
  #- accept *:991       # NAS Usenet
  #- accept *:992       # TELNETS
  - accept *:993       # IMAPS
  - accept *:995       # POP3S
  #- accept *:1194      # OpenVPN
  #- accept *:1293      # IPSec
  - accept *:3690      # SVN Subversion
  - accept *:4321      # RWHOIS
  - accept *:5222-5223 # XMPP, XMPP SSL
  - accept *:5228      # Android Market
  - accept *:9418      # git
  - accept *:11371     # OpenPGP hkp
  - accept *:64738     # Mumble
  - reject *:*
```


#### Abuse notices

During the lifetime of this relay, I have received two abuse notices.

On both occasions I was missing a reverse DNS pointer record (PTR) that would have made it more obvious that it's a Tor exit relay ([https://community.torproject.org/relay/setup/exit](https://community.torproject.org/relay/setup/exit))

| ![Abuse notice #1](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/1984-abuse-notice-1.png "Abuse notice #1") |
|:--:|
|Abuse notice #1 - CERT-IS - Emotet infection|

The first notice was from CERT-IS. The relay was likely involved in the distribution or C2 for the Emotet malware. I wasn't quick enough to respond and 1984 hosting suspended the VPS temporarily.

After an email to customer support, an updated PTR record and a quick security assessment the VPS was fully restored.

| ![Abuse notice #2](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/1984-abuse-notice-2.png "Abuse notice #2") |
|:--:|
|Abuse notice #2 - SK Telecom - HTTP intrusion attempt|

The second notice was from SK Telecom. Seemed the relay was routing traffic for someone trying to access something they shouldn't. This is why we can't have nice things.

I had recently reprovisioned the VPS and forgot to update the PTR record.

#### Monitoring with Nyx

Nyx ([https://nyx.torproject.org](https://nyx.torproject.org)) is a command-line monitor that shows real-time stats. Can be useful for troubleshooting. It is also just nice to watch sometimes.

| ![Nyx monitoring](/imgs/2022-06-01-Experiences-running-a-Tor-exit-node-for-2-years/tor-relay-nyx.png "Nyx monitoring") |
|:--:|
|Nyx monitoring|


#### Tor swag

If you run a fast relay, or have contributed to the project in some other way, you might be eligible for a Tor T-Shirt - ([https://community.torproject.org/relay/community-resources/swag](https://community.torproject.org/relay/community-resources/swag))

I am still waiting for mine. Will share an update when there is one.

#### Summary

Operating a relay has been a bit of fun. It doesn't cost much and there is no significant admin overhead. It is set and forget, for the most part.

If you have used Tor in the past, you should consider running a relay yourself. Operating a relay from your home or work place might not be a smart idea, so I would recommend checking out 1984 hosting.
