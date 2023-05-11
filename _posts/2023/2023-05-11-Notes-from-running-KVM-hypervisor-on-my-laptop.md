---
layout: post
title:  "Notes from running a KVM hypervisor on my laptop"
date:   '2023-05-11'
author: Dan Kir
tags:   
  - linux
  - virtualization
description: >-
  Notes from running a KVM hypervisor on my laptop
---

#### Compress QCOW2 disks and reclaim free space

Thin-provisioned QCOW2 disks will grow in size as required. They do not automatically shrink. I had some disks with over 100GB allocated on their host and less than 10GB in the filesystem.

```bash
## If disk is not sparse already, this will convert the disk in place.
virt-sparsify --in-place Virt/kali.qcow2

## I have a small /tmp partition. Set a temporary temp directory
export TMPDIR=/home/dan/tmp

## Create a compressed disk. Could take a long time depending on disk size.
virt-sparsify --compress disk.qcow2 disk-compressed.qcow2
```

#### Resize VM to window running XFCE4
Auto resizing VMs running XFCE4 does not work.

Can trigger the resize with the following.

```bash
##Identify the virtual monitor name
xrandr --listmonitors           
Monitors: 1
0: +*Virtual-1 1600/423x900/238+0+0  Virtual-1

## Trigger the resize
xrandr --output Virtual-1 --auto

```
In VMs running XFCE4 I like to create a launcher for this on my taskbar panel

|![Alt text](/imgs/2023-05-11-Notes-from-running-KVM-hypervisor-on-my-laptop/resize-launcher.png "XFCE4 Monitor Resize Launcher")|
|:--:|
|XFCE4 Monitor Resize Launcher|

#### Problem saving running memory to disk

On my laptop, I have allocated 16GB to my /var partition. All of my VM disks are in my /home directory.

The problem is when you choose to "Save" a running VM, the virtual memory is written to disk at `/var/lib/libvirt/qemu/save/`

I have not found a way to change this default behavior.


#### CPU topology Hashcat benchmarks

All tests done with Skylake-Client-IBRS model and CPU security flaw mitigations disabled.

My laptop CPU is a Xeon E3-1505M.

**1 Socket x 4 Cores**
```bash
hashcat --benchmark
-------------------
* Hash-Mode 0 (MD5)
-------------------
Speed.#1.........:   294.1 MH/s (2.97ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------
Speed.#1.........: 82801.3 kH/s (12.59ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

-----------------------
* Hash-Mode 1000 (NTLM)
-----------------------
Speed.#1.........:   390.9 MH/s (2.49ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

--------------------------------------------------------
* Hash-Mode 7500 (Kerberos 5, etype 23, AS-REQ Pre-Auth)
--------------------------------------------------------
Speed.#1.........:  1913.6 kH/s (68.26ms) @ Accel:32 Loops:1024 Thr:1 Vec:8
```

**4 Sockets x 1 Core**
```bash
hashcat --benchmark
-------------------
* Hash-Mode 0 (MD5)
-------------------
Speed.#1.........:   335.2 MH/s (2.97ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------
Speed.#1.........: 87031.4 kH/s (11.87ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

-----------------------
* Hash-Mode 1000 (NTLM)
-----------------------
Speed.#1.........:   539.8 MH/s (1.78ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

--------------------------------------------------------
* Hash-Mode 7500 (Kerberos 5, etype 23, AS-REQ Pre-Auth)
--------------------------------------------------------
Speed.#1.........:  1925.9 kH/s (67.86ms) @ Accel:256 Loops:128 Thr:1 Vec:8
```

**Results**

There is a noticeable performance increase when choosing multiple sockets over multiple cores.


#### UEFI Firmware means no snapshots

Heading is really self explanatory. I learnt the hard way that VMs using UEFI firmware can not currently be snapshotted.

#### Mounting shared folders

First configure the filesystem passthrough in Virt-Manager

|![Alt text](/imgs/2023-05-11-Notes-from-running-KVM-hypervisor-on-my-laptop/virt-manager-filesystem.png "Virt-Manager Filesystem Passthrough")|
|:--:|
|Virt-Manager Filesystem Passthrough|

Create a directory in the VM and mount

```bash
mkdir ShareFolder

sudo mount -t 9p -o trans=virtio /mnt/ShareFolder ShareFolder/
```
