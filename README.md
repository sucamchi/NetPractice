*This project has been created as part of the 42 curriculum by scamlett.*

# NetPractice

## Description

NetPractice is an introduction to computer networking. Simulated networks
are configured through a training interface that runs in the web browser. 
Each level shows a network diagram that does not work,
one or more objectives at the top of the page, and a set of editable fields.

The goal is to understand **TCP/IP addressing**, how a **switch** joins machines inside one
network and how a **router** connects separate networks.

## Instructions

From the repository root:

```sh
cd net_practice
./run.sh
```

The script picks a free port, starts a local Python web server, and tries to
open your browser. If it does not open by itself, visit the URL the script
prints.

If `run.sh` does not work on your machine, start the server yourself:

```sh
cd net_practice
python3 -m http.server 49242
```

Then open `http://localhost:49242` in a browser.

### Training workflow

1. Choose the **Training** tab and enter your 42 intranet login.
2. Read the objectives at the top of the level.
3. Edit the unshaded fields only.
4. Click **Check again**. The log panel at the bottom of the page traces the packet .
5. When every objective reports `OK`, click **Get my config** before moving on to download a small JSON file named after the level.
6. Click **Next level.** 

### Exporting and submitting

**Get my config** downloads one file per level, named `level1.json` through
`level10.json`. Each file records only the fields you were allowed to edit, for example:

```json
{"routes":{},"ifs":{"A1":{"ip":"104.95.23.11"},"B1":{},"C1":{},"D1":{"ip":"211.191.192.74"}}}
```

Submission requires 10 exported configuration files, one per level, placed at the root of your Git repository.


## Networking fundamentals

### What is a network?

A network is a group of devices that can exchange data. A **LAN** (local area
network) covers a limited area. The Internet
is an enormous network built out of many networks joined together.

For two devices to communicate, two things must exist: a physical or wireless
path between them, and an addressing scheme that identifies both the destination
device and the network it belongs to. NetPractice is entirely about the second
one.


Everything above the router is one subnet. Everything below it is another. The
router is the only device that belongs to both, which is exactly why traffic has
to pass through it to get from one to the other.

### The OSI and TCP/IP layer models

Networking is described in layers, where each layer relies on the one below it
and provides a service to the one above. Two models are in common use. The OSI
model has seven layers and is mostly a teaching tool. The TCP/IP model has four
and describes what really runs on the wire.

| TCP/IP layer | Job | OSI layers it covers | Examples |
| --- | --- | --- | --- |
| Application | Services programs actually use | Application, presentation, session | HTTP, DNS, SSH |
| Transport | Conversation between two programs | Transport | TCP, UDP |
| Internet | Addressing and routing between networks | Network | IP, ICMP |
| Link | Delivery across one physical hop | Data link, physical | Ethernet, Wi-Fi |

The two protocols in the name do different jobs:

- **IP** (Internet Protocol) gives every interface an address and works out
  which way a packet should travel. It makes no promise that the packet arrives.
- **TCP** (Transmission Control Protocol) builds a reliable connection on top of
  that. It splits data into numbered segments, checks what arrived, and asks for
  anything missing to be sent again.


### IPv4 addresses

An IPv4 address is a 32-bit number, written as four 8-bit blocks called octets,
separated by dots. Each octet holds a value from 0 to 255.

```
  192  .  168  .    1   .   10
11000000.10101000.00000001.00001010
```

Every address is really two fields glued together:

- The **network portion** says which network the device is on.
- The **host portion** says which device it is inside that network.

Nothing in the address itself says where the split falls. That is the subnet
mask's job, and it is why the same address can mean different things under
different masks.

Not every 32-bit value is a usable address:

- The **network address**, where every host bit is 0, names the subnet itself.
- The **broadcast address**, where every host bit is 1, means "everyone on this
  subnet".
- `127.0.0.0/8` is **loopback**, the machine talking to itself. It never appears
  on a link between two machines.
- Addresses from `224.0.0.0` upward are multicast and reserved, so the first
  octet of a normal host address stays at 223 or below.

IPv6 is the 128-bit successor to IPv4 that exists because 32 bits ran out and is not used in this project.

### Subnet masks and CIDR

A subnet mask is another 32-bit value, written the same way. Its bits are a run
of `1`s followed by a run of `0`s, and nothing else. The `1`s mark the network
portion, the `0`s mark the host portion.

```
255.255.255.0    11111111.11111111.11111111.00000000    24 network bits, 8 host bits
```

**CIDR notation** writes the same thing as a slash followed by the number of
network bits, so `255.255.255.0` and `/24` are two spellings of one mask. The
interface accepts either.

From the prefix, two numbers follow:

```
addresses in the block   = 2^(32 - prefix)
usable host addresses    = 2^(32 - prefix) - 2
```

The subtraction removes the network address and the broadcast address. This is
also why a `/31` has no usable hosts, and why the smallest subnet that can hold
a link between two routers is a `/30`, which gives exactly two.

### Finding a subnet range by hand

Binary works but is slow under exam conditions. This method gets you the same
answer in a few seconds, and it is the one worth drilling:

1. Find the octet where the mask is neither `255` nor `0`. Call its value *m*.
   This is the only octet that matters.
2. The **block size** is `256 - m`. If the octet is `0`, the block size is the
   whole octet, 256.
3. Count up from 0 in steps of the block size: `0`, `block`, `2 × block`, and so
   on. These are the boundaries.
4. Find the pair of boundaries your address sits between. That is your block.
5. The first address in the block is the **network address**, the last is the
   **broadcast address**, and everything in between is usable.


### CIDR reference table

Every mask you can meet, with the block size in the octet that varies. Learning
the bottom rows, `/24` down to `/30`, covers most of what the levels ask for.

| CIDR | Subnet mask | Block size (octet) | Addresses | Usable hosts |
| --- | --- | --- | --- | --- |
| /8 | 255.0.0.0 | 256 (2nd) | 16777216 | 16777214 |
| /9 | 255.128.0.0 | 128 (2nd) | 8388608 | 8388606 |
| /10 | 255.192.0.0 | 64 (2nd) | 4194304 | 4194302 |
| /11 | 255.224.0.0 | 32 (2nd) | 2097152 | 2097150 |
| /12 | 255.240.0.0 | 16 (2nd) | 1048576 | 1048574 |
| /13 | 255.248.0.0 | 8 (2nd) | 524288 | 524286 |
| /14 | 255.252.0.0 | 4 (2nd) | 262144 | 262142 |
| /15 | 255.254.0.0 | 2 (2nd) | 131072 | 131070 |
| /16 | 255.255.0.0 | 256 (3rd) | 65536 | 65534 |
| /17 | 255.255.128.0 | 128 (3rd) | 32768 | 32766 |
| /18 | 255.255.192.0 | 64 (3rd) | 16384 | 16382 |
| /19 | 255.255.224.0 | 32 (3rd) | 8192 | 8190 |
| /20 | 255.255.240.0 | 16 (3rd) | 4096 | 4094 |
| /21 | 255.255.248.0 | 8 (3rd) | 2048 | 2046 |
| /22 | 255.255.252.0 | 4 (3rd) | 1024 | 1022 |
| /23 | 255.255.254.0 | 2 (3rd) | 512 | 510 |
| /24 | 255.255.255.0 | 256 (4th) | 256 | 254 |
| /25 | 255.255.255.128 | 128 (4th) | 128 | 126 |
| /26 | 255.255.255.192 | 64 (4th) | 64 | 62 |
| /27 | 255.255.255.224 | 32 (4th) | 32 | 30 |
| /28 | 255.255.255.240 | 16 (4th) | 16 | 14 |
| /29 | 255.255.255.248 | 8 (4th) | 8 | 6 |
| /30 | 255.255.255.252 | 4 (4th) | 4 | 2 |
| /31 | 255.255.255.254 | 2 (4th) | 2 | 0 |
| /32 | 255.255.255.255 | 1 (4th) | 1 | 1 |


### Switches

A switch joins several devices into one network. It forwards traffic between the
ports plugged into it and has no IP address of its own in this project, which is
why it never appears as an editable field.

The important consequence is that **a switch does not create a boundary**. Every
device attached to the same switch has to be on the same subnet, with the same
mask, and with a distinct host address. A switch cannot move traffic between two
different IP networks, and it cannot act as a gateway. When a level puts three
hosts around a switch, it is asking you to fit all three into one range.

### Routers, interfaces and the default gateway

A router joins different networks. It has several **interfaces**, and each one
is a separate foot planted in a separate network with its own IP address and
mask.

The rule that governs every router in this project is that **its interfaces must
not overlap**. If two interfaces on one router covered ranges that intersect,
the router could not tell which way to send a packet destined for the shared
part, so the configuration is rejected. When you carve a range into subnets for
a router, check every pair of interfaces against each other, not just the one
you are editing.

When a device wants to send to an address that is not in any of its own subnets,
it cannot deliver the packet itself. It hands it to its **default gateway**, the
router that will forward it onward. For the handover to work:

- The gateway must be an address inside one of the sender's own subnets. A
  gateway the sender cannot reach directly is useless.
- It must be the exact IP of the neighbouring interface, not merely an address
  in the right range. The neighbour only accepts a packet addressed to it.
- It must be a usable host address, never a network or broadcast address.
- It cannot be the sender's own address.


## Resources

- 42 NetPractice en.subject.pdf
- [NetPractice guide by caroldaniel](https://github.com/caroldaniel/42sp-cursus-netpractice)
- [NetPractice guide by sarafreitas](https://github.com/SaraFreitas-dev/NetPractice/)
- [NetPractice article by imyzf](https://medium.com/@imyzf/netpractice-2d2b39b6cf0a)
- [NetPractice guide by lpaube](https://github.com/lpaube/NetPractice)
- [NetPractice: An Intro to IP Addresses and Subnets](https://www.youtube.com/watch?v=HQUw0CfQWAM&t=1097s)

AI was used to help structure the README.md and understand new concepts.