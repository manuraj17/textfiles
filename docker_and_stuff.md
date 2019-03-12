---
title: "Docker_and_stuff"
date: 2018-05-22T12:11:11+05:30
draft: true
---

Disclaimer: This post contains more of borrowed text from various parts of the
internet as I believe the author has done good enough explanation. Credits will
be given in references. Rest may be considered as my own writings. The aim of
this post is more towards a collective explanation of certain topic, borrowed
and not.

# Docker vs Virtual Machine
```
Docker itself is not a VM, so there is no double layer of OS. Docker is a tool
to run applications with settings that isolate them from other applications
running on the same OS kernel. Docker does include a VM with Docker for Windows
and Docker for Mac to run the Linux kernel so you can run Linux containers.
There is an option to run native Windows containers with Server 2016.
```
```
Docker originally used LinuX Containers (LXC), but later switched to runC
(formerly known as libcontainer), which runs in the same operating system as its
host. This allows it to share a lot of the host operating system resources.
Also, it uses a layered filesystem (AuFS) and manages networking.

AuFS is a layered file system, so you can have a read only part and a write part
which are merged together. One could have the common parts of the operating
system as read only (and shared amongst all of your containers) and then give
each container its own mount for writing.

So, let's say you have a 1 GB container image; if you wanted to use a full VM,
you would need to have 1 GB times x number of VMs you want. With Docker and AuFS
you can share the bulk of the 1 GB between all the containers and if you have
1000 containers you still might only have a little over 1 GB of space for the
containers OS (assuming they are all running the same OS image).

A full virtualized system gets its own set of resources allocated to it, and
does minimal sharing. You get more isolation, but it is much heavier (requires
more resources). With Docker you get less isolation, but the containers are
lightweight (require fewer resources). So you could easily run thousands of
containers on a host, and it won't even blink. Try doing that with Xen, and
unless you have a really big host, I don't think it is possible.

A full virtualized system usually takes minutes to start, whereas
Docker/LXC/runC containers take seconds, and often even less than a second.

There are pros and cons for each type of virtualized system. If you want full
isolation with guaranteed resources, a full VM is the way to go. If you just
want to isolate processes from each other and want to run a ton of them on a
reasonably sized host, then Docker/LXC/runC seems to be the way to go.
```
`https://stackoverflow.com/questions/46285198/run-docker-without-host-os`
`https://stackoverflow.com/questions/16047306/how-is-docker-different-from-a-normal-virtual-machine`

# No Base OS Image?
Here's how:
```C
// hello.c
#include <stdio.h>

int main() {
    printf("Hello, world!\n");
    return 0;
}
```

```
FROM debian
RUN apt-get update
RUN apt-get -y install gcc
ADD hello.c /hello.c
RUN gcc -static -o /hello hello.c

FROM scratch
COPY --from=0 /hello /hello
CMD ["/hello"]
```
Using multistage builds.

# Userland
```
On one conceptual level, the kernel is everything that runs at a "more
privileged" level of hardware protection. That would be like ring 0 on x86
processors, system mode on ARM, kernel mode on MIPS, supervisor mode on 68xxx,
etc. The kernel is usually interrupt-driven, either software interrupts (system
calls) or hardware interrupts (disk drives, network cards, hardware timers).

On that same conceptual level, "user land" is what runs in the least privileged
mode (ring 3 on x86 CPUs, user mode on ARM or MIPS, etc.). User land takes
advantage of the way that the kernel smooths over minor hardware differences,
presenting the same API to all programs. For instance, some wireless cards might
have extra control registers with respect to others, or contain more or less
on-board buffer for incoming packets. The driver code accounts for these
differences (sometimes by ignoring advanced or unusual features), and presents
the same socket API to all programs.

Some processors (e.g. x86, VAX, Alpha AXP) have more than two modes, but the
generic Unix architecture doesn't use the intermediate modes.

The programs and processes you see running in Unix or Linux or *BSD are the user
land. Since processes are pre-emptible, you actually never see the kernel run,
you just see side effects, like a read() system call returning, or a signal
handler function running.

To answer your specific question, in Unix, Linux, *BSD a "driver" is usually
some smallish piece of software that deals with the specific peculiarities of
some piece of hardware: a network card, a disk drive, a video card. The driver
software almost always neeeds to run in Ring 0/supervisor mode/kernel space in
order to have access to hardware interrupts, or the mapped memory of the
hardware or whatever. The driver takes care of specific hardware features, and
makes that hardware fit into the kernel code's standardized or conventionalized
view of how hardware should work. Therefore, running a driver in user land
requires the kernel to show a userland program things like mapped-in memory or
device registers or interrupts or other special features. That can be tricky, as
the special features a device might require don't easily fit in to the usual
Unix-style API presented to user land programs. Also, scheduling is an issue, as
user land programs don't typically respond to interrupts all that rapidly.
```

#### References
https://unix.stackexchange.com/questions/137820/whats-the-difference-of-the-userland-vs-the-kernel

# References
https://stackoverflow.com/questions/46285198/run-docker-without-host-os

