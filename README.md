*This project has been created as part of the 42 curriculum by sucamchi.*

# NetPractice

## Description

NetPractice is an introduction to computer networking. Instead of writing a
program, you configure ten simulated networks through a training interface that
runs in your web browser. Each level shows a network diagram that does not work,
one or more objectives at the top of the page, and a set of editable fields. You
change those fields until every objective reports `OK`.

The goal is to understand **TCP/IP addressing**,
 how a **switch** joins machines inside one
network, how a **router** connects separate networks, and how a **default
gateway** and a **routing table** decide where a packet goes next.

## Instructions

### Running the training interface

From the repository root:

```sh
cd net_practice
./run.sh
```

The script picks a free port, starts a local Python web server, and tries to
open your browser. If it does not open by itself, visit the URL the script
prints, which looks like `http://localhost:49242`. Press `Ctrl-C` to stop the
server.

If `run.sh` does not work on your machine, start the server yourself:

```sh
cd net_practice
python3 -m http.server 49242
```

Then open `http://localhost:49242` in a browser.

### Training workflow

1. Choose the **Training** tab and enter your 42 intranet login.
2. Read the objectives at the top of the level. They name the machines that
   have to reach each other.
3. Work the addressing out before typing.
4. Edit the unshaded fields only. Shaded fields are locked and are part of the
   puzzle, not the solution.
5. Click **Check again**. The log panel at the bottom of the page traces the
   packet hop by hop and shows in red where it stops.
6. When every objective reports `OK`, click **Get my config** before moving on.
   The browser downloads a small JSON file named after the level.
7. Click **Next level** and repeat.

### Exporting and submitting

**Get my config** downloads one file per level, named `level1.json` through
`level10.json`. Each file records only the fields you were allowed to edit:

```json
{"routes":{},"ifs":{"A1":{"ip":"104.95.23.11"},"B1":{},"C1":{},"D1":{"ip":"211.191.192.74"}}}
```

Submission requires 10 exported configuration files, one per level, placed at
the root of your Git repository. Keep the filenames the interface produces.
Only what is in your repository is evaluated, so check the names before you
finish.

### Evaluation mode

The **Evaluation** tab generates a fresh random configuration and gives you
three levels drawn at random with a limited amount of time
to solve them.

## The ten levels


| Level | What is on screen | The idea it adds |
| --- | --- | --- |
| 1 | Two isolated pairs of hosts | Two machines talk only when both addresses fall inside the same subnet |
| 2 | Two isolated pairs of hosts | Masks must be contiguous to be valid, `/30` means the same as a dotted mask, and some addresses are reserved and unusable |
| 3 | Three hosts around a switch | A switch does not split anything: everything attached to it is one single subnet |
| 4 | Hosts, a switch, a router with three interfaces | A router's interfaces must sit on separate subnets that do not overlap |
| 5 | Two hosts, one router, first routing tables | The default gateway, and what a single route entry is made of |
| 6 | A host, a switch, a router, the Internet | Public against private addresses, and the default route out to the Internet |
| 7 | Two routers back to back, no switch | Splitting one address range into several non-overlapping subnets, one per link |
| 8 | Two hosts, two routers, an ISP, the Internet | Several routes on one router, and a path that has to work end to end |
| 9 | Four hosts, two routers, a switch, the Internet | Everything at once, including the order routes are written in |
| 10 | The same scale, with most fields locked | Deducing the few values you may change from the many you may not |

## Networking fundamentals

### What is a network?

A network is a group of devices that can exchange data. A **LAN** (local area
network) covers a limited area such as a flat, a floor or a campus. The Internet
is an enormous network built out of many networks joined together.

For two devices to communicate, two things must exist: a physical or wireless
path between them, and an addressing scheme that identifies both the destination
device and the network it belongs to. NetPractice is entirely about the second
one.

Here is the shape almost every level ends up having:

```
  host A                 host B
  192.168.1.1            192.168.1.2
      |                      |
      +------- switch -------+
                 |
            R11: 192.168.1.254     <- same subnet as A and B, so it can be their gateway
        +--------+---------+
        |     router R1    |
        +--------+---------+
            R12: 163.172.250.12    <- a different subnet, facing outward
                 |
            Internet: 163.172.250.1
```

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

Three of these layers map directly onto things you see in the levels. A switch
works at the link layer, moving frames inside one network. A router works at the
Internet layer, choosing between networks. TCP sits above both and is the reason
a connection needs to work in **both directions**: a request that arrives but
whose reply cannot find its way home is not a working connection. Every
objective in NetPractice is checked in both directions for this reason.

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

Assigning any of these to an interface is invalid, and levels 2 and 3 hand you
exactly that mistake to find and fix.

IPv6 is the 128-bit successor to IPv4 and exists because 32 bits ran out. It is
not used in this project.

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

The ones-then-zeros rule is strict. `255.255.255.32` looks plausible and is not a
mask at all, because in binary it is `11111111.11111111.11111111.00100000`: the
run of ones is broken by zeros. Any mask with a gap is rejected.

From the prefix, two numbers follow:

```
addresses in the block   = 2^(32 - prefix)
usable host addresses    = 2^(32 - prefix) - 2
```

The subtraction removes the network address and the broadcast address. This is
also why a `/31` has no usable hosts, and why the smallest subnet that can hold
a link between two routers is a `/30`, which gives exactly two.

To find which network an address belongs to, apply a bitwise AND between the
address and the mask. A `1` in the mask keeps the address bit, a `0` clears it:

```
  192.168.20.222   11000000.10101000.00010100.11011110
& 255.255.255.224  11111111.11111111.11111111.11100000
= 192.168.20.192   11000000.10101000.00010100.11000000
```

Two interfaces are on the same subnet when this calculation gives them the same
answer, using the same mask. If either the address or the mask differs enough to
change the result, they cannot talk to each other directly, even with a cable
between them.

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

**Example 1.** `10.20.4.13` with mask `255.255.255.248`, that is `/29`.

```
Block size     256 - 248 = 8
Boundaries     0, 8, 16, 24, ...
13 sits in     8 to 15

Network        10.20.4.8
Usable         10.20.4.9  to  10.20.4.14
Broadcast      10.20.4.15
```

**Example 2.** `192.168.20.222` with mask `255.255.255.224`, that is `/27`.

```
Block size     256 - 224 = 32
Boundaries     0, 32, 64, ..., 192, 224
222 sits in    192 to 223

Network        192.168.20.192
Usable         192.168.20.193  to  192.168.20.222
Broadcast      192.168.20.223
```

**Example 3, where the block is not in the last octet.** `118.198.15.30` with
mask `255.255.254.0`, that is `/23`. The interesting octet is the third.

```
Block size     256 - 254 = 2, counted in the THIRD octet
Boundaries     0, 2, 4, ..., 14, 16
15 sits in     14 to 15

Network        118.198.14.0
Usable         118.198.14.1  to  118.198.15.254
Broadcast      118.198.15.255
```

This last case is the one that catches people out. A `/23` subnet spans two
values of the third octet, so `118.198.14.200` and `118.198.15.7` are neighbours
on the same network even though the octet in the middle differs.

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

The pattern in the mask column is worth noticing on its own. Each step down
halves the block, and the values `128, 192, 224, 240, 248, 252, 254, 255` are
the only ones that can appear in the octet that varies. If a mask in a level
contains any other number, it is invalid.

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

In the diagram earlier, host A's gateway is `192.168.1.254`, which is R11's
address. R11 is on A's subnet, so A can reach it, and from there R1 takes over.

### Routing tables

A routing table is the list a device consults when the destination is not on one
of its own subnets. Each entry has two parts:

- **Destination**: a network in CIDR form, such as `192.168.10.0/24`. It is
  matched against the destination address of the packet.
- **Next hop** (the gateway): the address of the device that should receive the
  packet next. The same rules as for a default gateway apply, since a default
  gateway is simply one of these entries.

The entry `0.0.0.0/0`, also written `default`, matches every possible address.
It is the catch-all for "anything I have no specific entry for", and it is
normally what points at the Internet.

**Order matters.** The table is read from top to bottom and the first entry
whose destination matches is the one used, so a `default` entry placed above a
specific one would swallow everything and the specific entry would never be
reached. Write specific routes first and keep `default` last.

Reading a table, then, is: take the destination address, test it against each
entry's network in turn, stop at the first match, and send the packet to that
entry's next hop. If nothing matches at all, the packet is dropped.

### Public and private addresses

Some ranges are reserved for use inside private networks. They are free for
anyone to use at home or in a company, and precisely because everyone reuses
them, **routers on the Internet refuse to carry them**.

| Range | CIDR | Use |
| --- | --- | --- |
| 10.0.0.0 to 10.255.255.255 | `10.0.0.0/8` | Private |
| 172.16.0.0 to 172.31.255.255 | `172.16.0.0/12` | Private |
| 192.168.0.0 to 192.168.255.255 | `192.168.0.0/16` | Private |
| 127.0.0.0 to 127.255.255.255 | `127.0.0.0/8` | Loopback, never on a link |
| 224.0.0.0 and above | `224.0.0.0/4` | Multicast and reserved |

Any level containing the Internet cares about this distinction. An interface
facing the Internet needs a public address, and traffic sent toward the Internet
with a private destination goes nowhere. Inside your own network, private
addresses are perfectly normal and are what you would usually choose.

## A method that works on every level

Work outward from the links you can see, and calculate before you type.

1. **Write down every interface**: its name, its IP and its mask. Do this for
   locked fields too, because those are the constraints that decide the rest.
2. **Calculate each subnet**: network address, usable range, broadcast address,
   using the block-size method above.
3. **Group the interfaces by link.** Two interfaces joined by a cable, or joined
   through a switch, must land in the same subnet with the same mask and
   different host addresses.
4. **Check the routers.** No two interfaces on one router may overlap. This is
   usually what forces you to use a longer prefix, splitting one range into
   several small ones.
5. **Set the gateways.** Each host's gateway is the router interface sitting on
   that host's own subnet.
6. **Fill in the routes** for networks that are not directly attached, next hop
   first, and keep `default` at the bottom.
7. **Walk the path by hand, in both directions.** Start at the source, ask at
   each device "is the destination on one of my subnets?", follow either the
   interface or the matching route, and repeat. Then do the same walk backwards,
   because the reply has to find its way home too.
8. **Click Check again and read the log.** It replays that same walk and marks
   in red the first hop that failed. Fix that hop and only that hop, then check
   again. Guessing at random tends to break the parts that already worked.

## Common mistakes

- Giving an interface the network or the broadcast address of its own subnet.
- Two interfaces on one subnet sharing the same IP address.
- Two interfaces on the same router covering overlapping ranges.
- A mask whose binary form is not an unbroken run of ones, such as
  `255.255.255.32`.
- A gateway that is not inside any of the sender's own subnets.
- A gateway that is in the right subnet but is not the exact address of the
  neighbouring interface.
- A `default` route written above a more specific route, which hides it.
- Sending private addresses toward the Internet, or putting a private address on
  an interface that faces it.
- Forgetting the return path. A configuration that delivers the request but
  cannot deliver the reply still fails.
- Leaving an address in `127.0.0.0/8` or above `223` in the first octet on a
  real link.

## Resources

- 42 NetPractice en.subject.pdf
- [NetPractice guide by caroldaniel](https://github.com/caroldaniel/42sp-cursus-netpractice)
- [NetPractice article by imyzf](https://medium.com/@imyzf/netpractice-2d2b39b6cf0a)
- [NetPractice guide by lpaube](https://github.com/lpaube/NetPractice),

AI was used to help structure the README.md and understand new concepts.