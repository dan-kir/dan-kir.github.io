---
layout: post
title:  "Experimenting with Hardened Malloc"
date:   '2022-05-22'
author: Dan Kir
tags:   
  - security
  - linux
description: >-
  Experimenting with Hardened Malloc
---

#### What is Hardened Malloc?

Hardened Malloc is a hardened memory allocator that provides substantial protection against heap corruption. It does this by adding memory guards, randomization and padding to allocated slabs.

I use Hardened Malloc everyday. It is loaded system wide on my phone (GrapheneOS) and most servers. I also use it selectively on my laptop workstation.

Below are some tests that demonstrate some of its capabilities and the additional memory requirements.

For how to compile/install the Hardened Malloc library see the references below.


#### Testing environment

```bash
dan@debian:~$ cat /etc/debian_version
11.3

dan@debian:~$ uname -a
Linux debian 5.10.0-13-amd64 #1 SMP Debian 5.10.106-1 (2022-03-17) x86_64 GNU/Linux

dan@debian:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.1Gi       1.5Gi        22Mi       1.2Gi       2.5Gi
Swap:          2.0Gi          0B       2.0Gi

dan@debian:~$ gcc --version
gcc (Debian 10.2.1-6) 10.2.1 20210110

dan@debian:~$ cat /proc/sys/kernel/randomize_va_space
2 # default
```

#### Testing program

```c
// gcc addresses.c -no-pie -fno-pie -ldl
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

int main()
{
    int stack;
    int *heap = malloc(sizeof(int));

    printf("executable: %p\n", &main);
    printf("stack: %p\n", &stack);
    printf("heap: %p\n", heap);
    printf("system@plt: %p\n", &system);

    void *handle = dlopen("libc.so.6", RTLD_NOW | RTLD_GLOBAL);
    printf("libc: %p\n", handle);
    printf("system: %p\n", dlsym(handle, "system"));

    free(heap);
    return 0;
}
```


#### Test 1 - randomize_va_space=0 + no hardened_malloc

###### Results:
Addresses are all the same each time the program runs.

```bash
dan@debian:~$ cat /proc/sys/kernel/randomize_va_space
0 #ASLR is disabled

dan@debian:~$ ./a.out
executable: 0x401172
stack: 0x7fffffffe20c
heap: 0x4052a0
system@plt: 0x401040
libc: 0x7ffff7fbc500
system: 0x7ffff7e39e50

dan@debian:~$ ./a.out
executable: 0x401172
stack: 0x7fffffffe20c
heap: 0x4052a0
system@plt: 0x401040
libc: 0x7ffff7fbc500
system: 0x7ffff7e39e50

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007ffff7fd0000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007ffff7bcb000)
	libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007ffff7b9b000)

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007ffff7fd0000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007ffff7bcb000)
	libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007ffff7b9b000)
```

#### Test 2 - randomize_va_space=1 + no hardened_malloc

###### Results:
Addresses for the stack and shared libraries have been randomized between runs. Other addresses remain unchanged.

```bash
dan@debian:~$ cat /proc/sys/kernel/randomize_va_space
1 #Stack and shared library offsets are randomized

dan@debian:~$ ./a.out
executable: 0x401172
stack: 0x7ffc6ce5510c
heap: 0x4052a0
system@plt: 0x401040
libc: 0x7f4302c89500
system: 0x7f4302b06e50

dan@debian:~$ ./a.out
executable: 0x401172
stack: 0x7ffc8ee29cec
heap: 0x4052a0
system@plt: 0x401040
libc: 0x7f182e4f1500
system: 0x7f182e36ee50

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007ffeb5732000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007f05bf60f000)
	libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007f05bf5df000)

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007ffe81d97000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007ffadd51a000)
	libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007ffadd4ea000)
```

#### Test 3 - randomize_va_space=2 + no PIE + no hardened_malloc

###### Results:
The executable and Procedure Linkage Table (PLT) addresses are consistent between runs.

```bash
dan@debian:~$ cat /proc/sys/kernel/randomize_va_space
2 #Heap offset is also randomized

dan@debian:~$ ./a.out
executable: 0x401172
stack: 0x7ffc84cdad5c
heap: 0xdb32a0
system@plt: 0x401040
libc: 0x7f88b98a5500
system: 0x7f88b9722e50

dan@debian:~$ ./a.out
executable: 0x401172
stack: 0x7ffcfb9959bc
heap: 0x12652a0
system@plt: 0x401040
libc: 0x7f4aba9ba500
system: 0x7f4aba837e50

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007ffd3e14e000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007f3c21761000)
	libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007f3c21731000)

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007ffdcffd5000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007fb9fc074000)
	libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007fb9fc044000)
```

#### Test 4 - randomize_va_space=2 + fPIE + no hardened_malloc

###### Results:
All addresses have been randomized

No guards or padding in heap.

Full TOR process memory maps {link here}

```bash
dan@debian:~$ cat /proc/sys/kernel/randomize_va_space
2 #Heap offset is also randomized

dan@debian:~$ ./a.out
executable: 0x558e7048b175
stack: 0x7fff89d8719c
heap: 0x558e70a572a0
system@plt: 0x7fabb2172e50
libc: 0x7fabb22f5500
system: 0x7fabb2172e50

dan@debian:~$ ./a.out
executable: 0x557e915b8175
stack: 0x7fff56371e6c
heap: 0x557e91cde2a0
system@plt: 0x7f0ed71fbe50
libc: 0x7f0ed737e500
system: 0x7f0ed71fbe50

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007ffdc5bf7000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007f6828efc000)
	libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007f6828ecc000)

dan@debian:~$ ldd /usr/bin/vi | head -n 3
	linux-vdso.so.1 (0x00007fff8b7d3000)
	liblua5.1-luv.so.0 => /lib/x86_64-linux-gnu/liblua5.1-luv.so.0 (0x00007f24dc974000)
    libuv.so.1 => /lib/x86_64-linux-gnu/libuv.so.1 (0x00007f24dc944000)

dan@debian:~$ sudo pmap 18276 | tail -n 1
  total      46348K

```

#### Test 5 - randomize_va_space=2 + hardened_malloc (default)

###### Results:
The executable and PLT addresses are the same

Guards, padding and additional randomization in heap.

```bash
dan@debian:~$ cat /proc/sys/kernel/randomize_va_space
2 #Heap offset is also randomized

dan@debian:~$ LD_PRELOAD="/usr/lib/libhardened_malloc.so/libhardened_malloc.so" ./a.out
executable: 0x401172
stack: 0x7ffe1b80448c
heap: 0x72ec09e61420
system@plt: 0x401040
libc: 0x7f5062db7a50
system: 0x7f5062c17e50

dan@debian:~$ LD_PRELOAD="/usr/lib/libhardened_malloc.so/libhardened_malloc.so" ./a.out
executable: 0x401172
stack: 0x7ffe1743427c
heap: 0x73451763d4f0
system@plt: 0x401040
libc: 0x7f96fc90ba50
system: 0x7f96fc76be50
```

#### Test 6 - randomize_va_space=2 + fPIE + hardened_malloc (default)

###### Results:
All addresses randomized

Guards, padding and additional randomization in heap.

Significant increase in memory usage.

Full TOR process memory map {link here}

```bash
dan@debian:~$ cat /proc/sys/kernel/randomize_va_space
2

dan@debian:~$ LD_PRELOAD="/usr/lib/libhardened_malloc.so/libhardened_malloc.so" ./a.out
executable: 0x557bb9a19175
stack: 0x7ffc3b312b3c
heap: 0x737ef80466c0
system@plt: 0x7fece36e0e50
libc: 0x7fece3880a50
system: 0x7fece36e0e50

dan@debian:~$ LD_PRELOAD="/usr/lib/libhardened_malloc.so/libhardened_malloc.so" ./a.out
executable: 0x562423f82175
stack: 0x7ffc4c3c943c
heap: 0x739167fed140
system@plt: 0x7fefe1398e50
libc: 0x7fefe1538a50
system: 0x7fefe1398e50

dan@debian:~$ sudo pmap 21274 | tail -n 1
 total      13334846328K

```

#### References
- https://github.com/GrapheneOS/hardened_malloc
- https://www.proggen.org/doku.php?id=security:memory-corruption:protection:aslr
- https://www.kicksecure.com/wiki/Hardened_Malloc
- https://www.whonix.org/wiki/Packages_for_Debian_Hosts
