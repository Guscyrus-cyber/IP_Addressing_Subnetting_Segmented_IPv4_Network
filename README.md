IP Addressing and Subnetting -Building and Testing a Segmented IPv4 Network

Introduction

This lab focuses on one of the most important foundations of networking: IP addressing and subnetting. In a real organization, devices are rarely placed into one large network. Networks are divided into smaller logical networks called subnets to organize systems, control traffic, improve performance, and strengthen security.

In this hands-on lab, a small segmented IPv4 network will be designed, configured, and tested. The lab will demonstrate how an IPv4 address identifies a host, how a subnet mask determines which portion represents the network and host, how devices determine whether another system is local or remote, and how subnetting can separate different groups of systems.

1\. What is an IP Address?

An IP address (Internet Protocol address) is a logical address assigned to a device participating in an IP network.

It allows the network to answer two fundamental questions:

Where is the device?\
Where should the packet be delivered?

For example:

192.168.10.25

A computer may use this address to communicate with other computers, routers, servers, printers, or Internet services.

An IP address is somewhat similar to a street address: the network needs an address to determine where traffic should go.

2\. What is IPv4?

IPv4 — Internet Protocol version 4 uses 32-bit addresses.

An IPv4 address contains four decimal numbers called octets:

192.168.10.25

Each octet represents 8 bits:

192 168 10 25

\| \| \| \|

8 bits 8 bits 8 bits 8 bits

8 + 8 + 8 + 8 = 32 bits

Each octet can range from:

0 – 255

Examples include:

10.0.0.15

172.16.5.20

192.168.1.100

IPv4 supports approximately 4.3 billion possible addresses, although many ranges are reserved or used for special purposes.

3\. What is a Subnet?

A subnet is a smaller logical network created from a larger IP network.

Suppose an organization has: IP address 192.168.10.0/24, Instead of putting every computer into the same network, the administrator could divide it into smaller networks.

For example:

192.168.10.0/26

192.168.10.64/26

192.168.10.128/26

192.168.10.192/26

These could represent:\
\
Subnet 1 → Employees\
Subnet 2 → Servers\
Subnet 3 → Security systems\
Subnet 4 → Guest devices\
\
This process is called subnetting.

Subnetting is particularly important to cybersecurity because segmentation can help limit unnecessary communication between systems.

4\. What does /24 mean?

We will frequently see an address such as: 192.168.1.10/24\
The /24 is called the CIDR prefix length. It tells us that the first 24 bits identify the network.

A /24 corresponds to:

255.255.255.0

Conceptually:

IP Address: 192.168.1.10

Subnet Mask: 255.255.255.0

CIDR: /24

Network: 192.168.1.0

Host: .10

5\. What is IPv6?

IPv6 — Internet Protocol version 6 was developed partly to overcome IPv4 address exhaustion.

IPv6 uses 128-bit addresses, compared with IPv4's 32 bits.

An IPv6 address might look like: 2001:db8:1234:5678::25\
Instead of four decimal octets, IPv6 addresses are normally represented using hexadecimal groups separated by colons.

The address space is enormous:

IPv4 → 32 bits

IPv6 → 128 bits

IPv6 also changes several networking behaviors. For example, IPv6 uses Neighbor Discovery Protocol (NDP) rather than IPv4 ARP for neighbor discovery.

This particular lab will concentrate primarily on IPv4 subnetting, but later network labs can examine IPv6 traffic directly.

Security: What Is the Attack Surface in IP and subnet?

IP addressing and subnetting are not themselves vulnerabilities. However, the way networks are addressed, segmented, routed, and exposed creates an attack surface.

An attacker may first try to understand the network:

What IP addresses exist?

Which hosts respond?

Which subnet am I in?

What other subnets exist?

Where is the gateway?

Which ports are accessible?

Can one network reach another?

Some important risks include:

| Area | Example Security Concern |
|----|----|
| IPv4 | Host discovery, scanning, IP spoofing, exposed services |
| IPv6 | Rogue Router Advertisements, NDP abuse, overlooked IPv6 exposure |
| Subnets | Poor segmentation allowing unnecessary lateral movement |
| Routing | Incorrect routes exposing protected networks |
| Addressing | Misconfiguration or address conflicts |
| ICMP | Network reconnaissance and host discovery |
| Network services | Attackers discovering listening ports and services |

One important SOC concept is lateral movement.

Imagine:

Internet

\|

Firewall

\|

Employee Network

\|

Compromised PC

\|

X

\|

Server Network

If segmentation and security controls restrict communication between the employee and server networks, compromising one workstation does not automatically mean unrestricted access to every server. That is one reason subnetting becomes much more than a mathematical networking exercise.

For this lab, I am using a virtual network simulation rather than modifying my MacBook's actual home network configuration.

The conceptual environment will look similar to:

Router

/ \\

/ \\

Subnet A Subnet B

192.168.10.0/26 192.168.10.64/26

\| \|

Host A Host B

I will work through:

IPv4 address identification → subnet masks → CIDR → network/broadcast addresses → usable host ranges → subnet design → host configuration → connectivity testing → segmentation verification → packet analysis/security observations.

Step 1: Inspect the Current IPv4 Configuration

I open Terminal and I run: ifconfig en0

There are three pieces of information to understand before constructing the segmented lab:

inet → IPv4 address

netmask → subnet mask

broadcast → broadcast address

inet 192.168.1.193 netmask 0xffffff00 broadcast 192.168.1.255

From your output: inet 192.168.1.193 netmask 0xffffff00 broadcast 192.168.1.255

| Field | Your value | Meaning |
|----|----|----|
| IPv4 address | 192.168.1.193 | Your Mac's private IPv4 address |
| Netmask | 0xffffff00 | Hexadecimal form of 255.255.255.0 |
| CIDR | /24 | 24 bits identify the network |
| Network address | 192.168.1.0 | Identifies the subnet |
| Broadcast | 192.168.1.255 | Broadcast address for the subnet |
| Usable range | 192.168.1.1–192.168.1.254 | Potential host addresses |

So, my Mac is currently on:

192.168.1.0/24

and my Mac is host:

192.168.1.193 which is the private IP address. (Images 1 and 2)

\
\
\
Step 2 — Understand the /24 Subnet
----------------------------------

why the current network is /24.

I Run: ipconfig getoption en0 subnet_mask

command returned: 255.255.255.0

That is the decimal subnet mask corresponding to: 255.255.255.0 = /24

### What /24 means

IPv4 contains 32 bits. With /24, the first 24 bits identify the network, while the remaining 8 bits identify individual hosts:

192 . 168 . 1 . 193

───────────── ───

Network Host

24 bits 8 bits

Using the information from Steps 1 and 2:

Mac IPv4: 192.168.1.193

Subnet mask: 255.255.255.0

CIDR: /24

Network address: 192.168.1.0

Broadcast: 192.168.1.255

Usable hosts: 192.168.1.1 – 192.168.1.254 (Image 3)

\
\
Step 3 — Find the Default Gateway

Identifying the router/default gateway that allows the Mac to leave its local subnet.

I run: route -n get default \| grep gateway

Your result is: gateway: 192.168.1.1

That means the current network looks like this:

Local Network: 192.168.1.0/24

MacBook Router

192.168.1.193 ─────────────────────────► 192.168.1.1

/24 Default Gateway

255.255.255.0

│

▼

Internet

The default gateway is the router my Mac sends packets to when their destination is outside the local subnet. (Image 4)\

\
\
\
Step 4 — Test Communication Inside the Subnet
---------------------------------------------

Communicating my Mac with its default gateway.

I run: ping -c 4 192.168.1.1

This sends four ICMP Echo Requests from my Mac (192.168.1.193) toward the router (192.168.1.1).

If communication succeeds, I'll see replies similar to:

64 bytes from 192.168.1.1: icmp_seq=0 ...

This is important because both addresses belong to: 192.168.1.0/24

So, the Mac can reach the router directly on its local subnet; it doesn't need another router to reach 192.168.1.1.\
\
The result:

4 packets transmitted

4 packets received

0.0% packet loss

confirms successful ICMP communication between my MacBook and its default gateway.

The four lines:

64 bytes from 192.168.1.1 are the router's ICMP Echo Replies.

It has established:

Network: 192.168.1.0/24

MacBook: 192.168.1.193

Subnet Mask: 255.255.255.0

Default Gateway: 192.168.1.1

Broadcast: 192.168.1.255

MacBook ── ICMP Echo Request ──► Router

MacBook ◄── ICMP Echo Reply ──── Router

Result: 4/4 successful (Image 5)

\
\
Step 5 A— Now We Start Subnetting
---------------------------------

I am going to take a /24 network and learn how to divide it into smaller networks. I will calculate it manually first, because understanding the calculation is an important networking/SOC skill.

For example, an organization owns this internal network: 192.168.10.0/24

It needs four separate network segments:\
\
Employees\
Servers\
Security/SOC\
Guest

I need to divide /24 into 4 equal subnets.

Starting with: 192.168.10.0/24 and I borrow 2 host bits:

2² = 4 subnets

Therefore:

/24 + 2 = /26

And /26 equals: 255.255.255.192

Each /26 contains 64 total addresses, giving these four subnets:

Employees 192.168.10.0/26

Servers 192.168.10.64/26

Security/SOC 192.168.10.128/26

Guest 192.168.10.192/26

why the subnet addresses jump by 64 (0 → 64 → 128 → 192) and how we determine the usable host and broadcast address for each subnet.

## Step 5 B— Calculate the Four /26 Subnets

Now I’ll see why the addresses increase by 64.

A /26 subnet mask is: 255.255.255.192

And the last octet: 256 - 192 = 64

So, each subnet contains 64 addresses, and the next network begins every 64 addresses:

0 → 64 → 128 → 192

Therefore the four segments are:

| Segment      | Network           | Usable Host Range | Broadcast |
|--------------|-------------------|-------------------|-----------|
| Employees    | 192.168.10.0/26   | .1 – .62          | .63       |
| Servers      | 192.168.10.64/26  | .65 – .126        | .127      |
| Security/SOC | 192.168.10.128/26 | .129 – .190       | .191      |
| Guest        | 192.168.10.192/26 | .193 .254         | .255      |

### Why only 62 usable hosts?

A /26 leaves:

32 - 26 = 6 host bits

Therefore:

2⁶ = 64 total addresses

64 - 2 = 62 usable host addresses

I subtract two because each subnet reserves:

First address → Network address

Last address → Broadcast address

For example, the Servers subnet is:

Network: 192.168.10.64

First host: 192.168.10.65

...

Last host: 192.168.10.126

Broadcast: 192.168.10.127

\
Step 6 — Verify a /26 Subnet Mathematically
-------------------------------------------

verifying the Servers subnet: 192.168.10.64/26\
So, there is\
\
/26 = 255.255.255.192 and 256 - 192 = 64 addresses per subnet

Therefore the block containing .64 runs:

Network: 192.168.10.64

First host: 192.168.10.65

Last host: 192.168.10.126

Broadcast: 192.168.10.127

Now let’s have the Mac calculate the address range so we have practical evidence rather than only doing it on paper.

I run this single command:

python3 -c 'import ipaddress; n=ipaddress.ip_network("192.168.10.64/26"); print("Network:",n.network_address); print("Netmask:",n.netmask); print("First host:",n.network_address+1); print("Last host:",n.broadcast_address-1); print("Broadcast:",n.broadcast_address); print("Total addresses:",n.num_addresses)'

Now after running that command, my Mac's calculation exactly matches the manual subnet calculation.

Result confirms:

Servers Subnet: 192.168.10.64/26

Network: 192.168.10.64

Subnet Mask: 255.255.255.192

First Host: 192.168.10.65

Last Host: 192.168.10.126

Broadcast: 192.168.10.127

Total Addresses: 64 (Image 6)

\
\
\
Step 7 — Build the Segmented IPv4 Network
=========================================

I need separate virtual hosts/networks so that I can actually demonstrate:

Employee subnet

│

├──── Router ──── Server subnet

│

├─────────────── SOC subnet

│

└─────────────── Guest subnet

I run which docker\
Docker basically is allowing to create arbitrary, isolated IPv4 subnets on my MacBook without changing or disrupting my real home network.\
\
For example, my real network remains:

Real Wi-Fi:

192.168.1.0/24

│

└── MacBook: 192.168.1.193

Inside Docker I am building:

Virtual Lab Network: 192.168.10.0/24

│

├── Employees: 192.168.10.0/26

├── Servers: 192.168.10.64/26

├── SOC: 192.168.10.128/26

└── Guest: 192.168.10.192/26

Next, I'll put virtual hosts (containers) into these subnets and assign them IP addresses. That lets me perform real networking tests such as checking IP configuration, pinging hosts, observing which hosts can communicate, testing cross-subnet behavior, examining gateways, and eventually seeing the traffic.

So in general docker gives us an actual controlled networking environment where the subnetting concepts can be tested safely.

So, Docker virtual network is an actual functioning network environment used to generate and observe network behavior. (Image 7)

Top of Form

Step 8 — Verify Docker Is Running\
\
I open Docker Desktop and wait until it finishes starting.

Then in terminal I run: docker info\
\
Docker is now running correctly. The Server section is present and shows:

Server Version: 29.6.1

Operating System: Docker Desktop

Architecture: aarch64

Containers: 4

Running: 3

That confirms the Docker engine is active. (Images 8, 9, and 10)

\
\
\

Step 9— Create the First Segmented IPv4 Network

Now creating the first actual lab subnet:

Employees subnet

192.168.10.0/26

I run:

docker network create \\

--driver bridge \\

--subnet 192.168.10.0/26 \\

employees_net

If successful, Docker will return a long network ID.

Then immediately I verify it with: docker network inspect employees_net

### What this does

Docker: Create an isolated Layer-3 IPv4 network\
Network: 192.168.10.0\
Prefix: /26\
Mask: 255.255.255.192\
Name: employees_net

This is now a real virtual subnet inside Docker.

The important section is:

"Driver": "bridge",

"EnableIPv4": true,

"EnableIPv6": false,

"Subnet": "192.168.10.0/26",

"Gateway": "192.168.10.1"

So Docker has created a real isolated IPv4 network:

employees_net

│

├── Network: 192.168.10.0/26

├── Mask: 255.255.255.192

├── Gateway: 192.168.10.1

└── Driver: bridge

Docker automatically assigned .1 as the virtual gateway, so .1 is already occupied. That's why Docker reports fewer dynamically available addresses than the theoretical 62 usable host addresses. (Image 11)

\
\
Step 10 — Create the Other Three Subnets
----------------------------------------

Now implementing the other three /26 networks from the design.

I run:

docker network create --driver bridge --subnet 192.168.10.64/26 servers_net

docker network create --driver bridge --subnet 192.168.10.128/26 soc_net

docker network create --driver bridge --subnet 192.168.10.192/26 guest_net

Each successful command should return a network ID.

Then I run: docker network ls

employees_net

servers_net

soc_net\
guest_net\
\
These confirm that all four virtual network segments were created successfully.

The lab now has:

Original network: 192.168.10.0/24

│

┌─────────────┼─────────────┐

│ │ │

▼ ▼ ▼

Employees Servers SOC Guest

192.168.10.0/26 192.168.10.64/26 192.168.10.128/26 192.168.10.192/26

And docker network ls confirms:

employees_net

servers_net

soc_net

guest_net (Image 12)\
\

\
Step 11 — Putting the First Host Inside the Employee Subnet
-----------------------------------------------------------

Now, there is networks but no lab computers inside them.

I create the first virtual workstation and give it a specific IPv4 address:

Employee-PC

IP: 192.168.10.10

Network: 192.168.10.0/26

I run:

docker run -dit \\

--name employee-pc \\

--network employees_net \\

--ip 192.168.10.10 \\

alpine sh

Then I verify the virtual host's address:

docker exec employee-pc ip addr show eth0

I am looking for:

inet 192.168.10.10/26

That /26 is especially important: it proves the virtual workstation has been placed inside our first subnet with the subnetting configuration we designed.\
\
The important result is:

inet 192.168.10.10/26 brd 192.168.10.63

So employee-pc is correctly running inside employees_net with:

IPv4: 192.168.10.10\
Prefix: /26\
Broadcast: 192.168.10.63\
Interface: eth0 (Image 13)\
\
\
Step 12 — Create and Verify Hosts in the Other Segmented Subnets
----------------------------------------------------------------

Already there is :

Employees subnet: 192.168.10.0/26

employee-pc: 192.168.10.10/26

Now I'll place one virtual host in each of the remaining three subnets:

Servers: server-01 → 192.168.10.70/26

SOC: soc-pc → 192.168.10.140/26

Guest: guest-pc → 192.168.10.200/26

These addresses all fall within the usable ranges which calculated earlier.

### 12A — Create the Server Host

I run:

docker run -dit \\

--name server-01 \\

--network servers_net \\

--ip 192.168.10.70 \\

alpine sh

Then I verify it:

docker exec server-01 ip addr show eth0

Expecting to see:

inet 192.168.10.70/26\
\
It confirms:

server-01

IPv4: 192.168.10.70/26

Broadcast: 192.168.10.127

Interface: eth0

Status: UP

This is for the Servers subnet 192.168.10.64/26.

The difference already:

employee-pc → 192.168.10.10/26 → broadcast 192.168.10.63

server-01 → 192.168.10.70/26 → broadcast 192.168.10.127

Even though both addresses begin with 192.168.10, the /26 mask places them in different subnets. (Image 14)

### 12B — Create the SOC Host

I run:

docker run -dit \\

--name soc-pc \\

--network soc_net \\

--ip 192.168.10.140 \\

alpine sh

Then: I run: docker exec soc-pc ip addr show eth0

Expecting : inet 192.168.10.140/26\
\
This confirms:

soc-pc

IPv4: 192.168.10.140/26

Broadcast: 192.168.10.191

Interface: eth0

Status: UP

That is correct for the SOC subnet: 192.168.10.128/26

So far:

Employees: employee-pc → 192.168.10.10/26

Servers: server-01 → 192.168.10.70/26

SOC: soc-pc → 192.168.10.140/26 (Image 15)

\
\
Step 12 C — Create the Guest Host
---------------------------------

I run:

docker run -dit \\

--name guest-pc \\

--network guest_net \\

--ip 192.168.10.200 \\

alpine sh

Then I verify: docker exec guest-pc ip addr show eth0

I am expecting result:

inet 192.168.10.200/26

brd 192.168.10.255

Output confirms:

guest-pc

IPv4: 192.168.10.200/26

Broadcast: 192.168.10.255

Interface: eth0

Status: UP

Now there are four separate /26 segments:

| Segment   | Subnet            | Host        | Host IP        |
|-----------|-------------------|-------------|----------------|
| Employees | 192.168.10.0/26   | employee-pc | 192.168.10.10  |
| Servers   | 192.168.10.64/26  | server-01   | 192.168.10.70  |
| SOC       | 192.168.10.128/26 | soc-pc      | 192.168.10.140 |
| Guest     | 192.168.10.192/26 | guest-pc    | 192.168.10.200 |

Together, the 13A–13C outputs provide strong evidence that the four hosts were placed into the intended subnets. (Image 16)

\
\
Step 13 — Test Network Segmentation
-----------------------------------

Now testing whether a host in one subnet can directly communicate with a host in another subnet.

From inside employee-pc, ping server-01:\
\
I run: docker exec employee-pc ping -c 4 192.168.10.70

Here is what makes this test important:

employee-pc

192.168.10.10/26

│

│ Different subnet

X

│

192.168.10.70/26

server-01

Because these are separate Docker bridge networks,I am expect them not to communicate directly unless routing between the networks is explicitly provided.

employee-pc (192.168.10.10/26)

│

│ ping → 192.168.10.70

X

│

server-01 (192.168.10.70/26)

4 transmitted

0 received

100% packet loss

This demonstrates that the Employee and Server hosts are on separate Docker bridge networks and currently have no inter-network routing path connecting them.\
The 100% packet loss is useful evidence here, not an error in the lab. (Image 17)

Bottom of Form

\
\
\
Step 14 — Test Same-Subnet Communication
----------------------------------------

Now creating a second Employee workstation inside the same 192.168.10.0/26 subnet.

Currently:\
\
employees_net — 192.168.10.0/26\
employee-pc\
192.168.10.10

I add:

employee-pc2

192.168.10.20

Both addresses are within the usable range 192.168.10.1 and 192.168.10.62

### Step 14A — Create the Second Employee Host

I run:

docker run -dit \\

--name employee-pc2 \\

--network employees_net \\

--ip 192.168.10.20 \\

alpine sh

Then I verify its address: docker exec employee-pc2 ip addr show eth0

I should see: inet 192.168.10.20/26

The output confirms:

employee-pc2

IPv4: 192.168.10.20/26

Broadcast: 192.168.10.63

Interface: eth0

Status: UP

Both Employee machines are now on exactly the same subnet:

employees_net — 192.168.10.0/26

employee-pc → 192.168.10.10/26

employee-pc2 → 192.168.10.20/26 (Image 18)

### \
\
Step 14B — Ping Between the Two Employee Hosts

From employee-pc, ping the new host:

docker exec employee-pc ping -c 4 192.168.10.20

expecting successful replies because both hosts belong to the same subnet:

employees_net

192.168.10.0/26

employee-pc employee-pc2

192.168.10.10 ◄──────────► 192.168.10.20

ICMP

Ideally, the result will show:

4 packets transmitted

4 packets received

0% packet loss

This gives a very useful comparison with Step 13:

Same subnet → communication succeeds

Different subnet → communication fails without inter-subnet routing

The result shows:

4 packets transmitted

4 packets received

0% packet loss

So there are now experimentally demonstrated the difference:

SAME SUBNET

employee-pc employee-pc2

192.168.10.10/26 ───────► 192.168.10.20/26

✅ SUCCESS

DIFFERENT SUBNETS

employee-pc server-01

192.168.10.10/26 ────X──► 192.168.10.70/26

❌ 100% packet loss

The reason is important: the two Employee containers share the same Docker bridge/subnet, while the Employee and Server containers are on separate Docker bridges and I have not configured a router between those two virtual networks. (Image 19)\
\

\
Step 15 — Examine the Host's Routing Table
------------------------------------------

Now let's see how employee-pc decides where packets should go.

I run: docker exec employee-pc ip route

Expecting:

default via 192.168.10.1 dev eth0

192.168.10.0/26 dev eth0 ...

This will connect three concepts:

IP address → subnet → default gateway/routing decision.

The routing table shows:

default via 192.168.10.1 dev eth0

192.168.10.0/26 dev eth0 scope link src 192.168.10.10

This means employee-pc knows two important things:

192.168.10.0/26 is directly connected through eth0. That's why it could communicate directly with 192.168.10.20.

For an address outside that subnet, the host sends the traffic toward its default gateway 192.168.10.1. However, the separate Docker bridge networks are not configured with an inter-subnet router that forwards traffic between them, which is why the earlier Employee → Server test failed. (Image 20)\
\
\
\
Step 16 — Final Verification and Security Conclusions
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

This is the final step of IP Addressing and Subnetting.

I am going to verify the finished topology in one command and then record what the lab proved.

I run:

docker ps --format "table {{.Names}}\t{{.Networks}}\t{{.Status}}"

It should be the lab containers associated with the networks, including:

employee-pc

employee-pc2

server-01

soc-pc

guest-pc

and networks such as:

employees_net

servers_net

soc_net

guest_net

The final output clearly verifies the lab topology:

employee-pc2 → employees_net

employee-pc → employees_net

server-01 → servers_net

soc-pc → soc_net

guest-pc → guest_net

The three single-node-wazuh... containers belong to the other Wazuh lab I did previously on my macbook environment and are unrelated to this networking lab. They don't invalidate the output.

### Lab conclusion 

This lab demonstrated IPv4 addressing and subnetting by dividing a /24 network into four /26 network segments for Employees, Servers, SOC, and Guest systems. Docker bridge networks and Alpine Linux containers were used to create an isolated virtual networking environment. IPv4 addresses, subnet masks, network ranges, broadcast addresses, and default gateways were examined and verified. Same-subnet ICMP communication succeeded, while communication between isolated Docker subnets failed without inter-subnet routing. The lab demonstrated how subnetting organizes network resources and how segmentation, when combined with routing and security controls, can restrict unnecessary communication and reduce opportunities for lateral movement. (Image 21)

Top of Form

Bottom of Form

### \
\
