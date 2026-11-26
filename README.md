# Multi-Site Enterprise Network Lab (Cisco Packet Tracer)

A two-site enterprise network built in Cisco Packet Tracer. It combines Layer 2 switching (VLANs, VTP, EtherChannel, STP, port security) with Layer 3 routing (router-on-a-stick, EIGRP, default and backup routes to an ISP), and adds DHCP, DNS and TFTP configuration backup.

> **Origin:** The topology and the base requirements come from a practice exercise in my Computer Networking course at Business and Technology University (BTU). The configuration, testing and the DHCP, DNS and TFTP backup additions are my own work.

## Skills Demonstrated

VLANs and inter-VLAN routing · VTP (server, client, transparent) · EtherChannel (LACP, PAgP, static) · STP root bridge planning · CDP control · Port security · EIGRP · Static, default and summary routes · DHCP on Cisco routers · DNS · TFTP configuration backup · VLSM subnetting · Troubleshooting with `show` commands

## Topology

<img width="1937" height="955" alt="image" src="https://github.com/user-attachments/assets/3e4fc200-1e53-4d75-8634-c7f1364f79cb" />

| File | Description |
|------|-------------|
| `EtherChannel-CDP-STP-VTP-EIGRP.pkt` | Packet Tracer project file |
| `configs/` | Saved running-config of each device (TFTP backup) |

## Addressing Plan

LANs and VLANs use `172.16.0.0/16` (VLSM). Point-to-point links use `200.0.0.0`.

### Site 1 (R3): VLANs 10-50, `172.16.2.0/24`

| VLAN | Name | Hosts | Network | Gateway (R3 subinterface) |
|------|------|-------|---------|---------------------------|
| 10 | test 1 | 20 | 172.16.2.224/27 | 172.16.2.225 (Gi0/1.10) |
| 20 | test 2 | 40 | 172.16.2.128/26 | 172.16.2.129 (Gi0/1.20) |
| 30 | test 3 | 30 | 172.16.2.192/27 | 172.16.2.193 (Gi0/1.30) |
| 40 | test 4 | 50 | 172.16.2.64/26 | 172.16.2.65 (Gi0/1.40) |
| 50 | Management | 60 | 172.16.2.0/26 | 172.16.2.1 (Gi0/1.50) |
| 60 | Native | - | - | - |
| 70 | Unused | - | - | - |

### Site 2 (R4): VLANs 100-500, `172.16.0.0/23`

| VLAN | Name | Hosts | Network | Gateway (R4 subinterface) |
|------|------|-------|---------|---------------------------|
| 100 | branch1 | 20 | 172.16.1.160/27 | 172.16.1.161 (Gi0/1.100) |
| 200 | branch2 | 80 | 172.16.1.0/25 | 172.16.1.1 (Gi0/1.200) |
| 300 | branch3 | 160 | 172.16.0.0/24 | 172.16.0.1 (Gi0/1.300) |
| 400 | branch4 | 30 | 172.16.1.128/27 | 172.16.1.129 (Gi0/1.400) |
| 500 | Management | 10 | 172.16.1.192/28 | 172.16.1.193 (Gi0/1.500) |
| 600 | Native | - | - | - |
| 700 | Unused | - | - | - |

### Router Links

| Link | Subnet | Addresses |
|------|--------|-----------|
| R1 - R2 | 200.0.0.0/30 | R1 Se0/0/1 200.0.0.1, R2 Se0/0/0 200.0.0.2 |
| R1 - R3 | 200.0.0.4/30 | R1 Se0/0/0 200.0.0.5, R3 Se0/0/0 200.0.0.6 |
| R3 - R4 | 200.0.0.8/30 | R3 Se0/0/1 200.0.0.9, R4 Se0/0/0 200.0.0.10 |
| R2 - ISP (Ethernet) | 13.0.0.0/30 | R2 Gi0/1 13.0.0.1 |
| R2 - ISP (serial, backup) | 14.0.0.0/30 | R2 Se0/0/1 14.0.0.1 |
| ISP - Server | 111.0.0.0/30 | Server1 (google.com) 111.0.0.2 |

R1 also has three loopback interfaces that simulate additional LANs (`10.0.4.1`, `10.0.0.1`, `10.0.6.1`).

### Servers

| Server | Role | Address | Network |
|--------|------|---------|---------|
| Server0 | DNS | 172.16.1.200 | Site 2, VLAN 500 |
| TFTP | Configuration backup | 172.16.2.20 | Site 1, VLAN 50 |
| Server1 | google.com (ISP side) | 111.0.0.2 | ISP |

## What Was Configured

### Layer 2 (switching)

- **VLANs:** Each branch or department has its own VLAN, so traffic is separated. Site 1 uses VLANs 10-50 (plus 60 as the native VLAN and 70 as an unused VLAN). Site 2 uses VLANs 100-500 (plus 600 native and 700 unused). Each subnet is sized with VLSM to fit its number of hosts.
- **VTP:** VLANs are created on the VTP server and shared automatically with the VTP clients (VTP version 2, domain `R3`). One switch at Site 2 runs in transparent mode, so it keeps its own VLAN list and does not take VLANs from the server.
- **EtherChannel:** Several physical links are combined into one logical link for more bandwidth and for redundancy. I used LACP (open standard), PAgP (Cisco) and one static channel with no dynamic protocol, as the exercise required.
- **STP:** STP prevents loops between switches. I chose the root bridges manually so traffic follows a planned path. Site 1 has one root bridge for all VLANs and a secondary root as backup. Site 2 splits the VLANs between two roots: R4S3 for VLANs 100-400 and R4S2 for VLANs 1 and 500-700, with R4S1 and R4S4 as their secondaries.
- **CDP:** CDP shares device information with neighbors. I turned it off on the ISP links, on the link to R3 and on EtherChannel 2, so less information is exposed.
- **Port security:** Every switch port limits which devices can connect. Site 1: maximum 2 MAC addresses, sticky learning, violation mode `restrict`. Site 2: maximum 3 MAC addresses, sticky learning, violation mode `protect`. Unused ports are shut down.

### Layer 3 (routing and services)

- **Inter-VLAN routing:** R3 and R4 use router-on-a-stick. One physical link carries all VLANs, and each VLAN has its own subinterface that acts as its default gateway.
- **EIGRP AS 100:** Routers share their networks automatically. LAN-facing interfaces are passive, so no routing messages are sent to hosts. `network` statements use wildcard masks, the metric uses delay only, and the hello interval is 3 seconds for faster neighbor detection.
- **ISP routing:** The network has a default route towards the ISP. The ISP has one static summary route that covers all the LANs. A backup static route goes over the serial link, so if the Ethernet link to the ISP fails, traffic can still get out.
- **DHCP (my addition):** R3 and R4 act as DHCP servers with one pool per VLAN. The first six addresses of every subnet are excluded for the gateway and static devices. Clients automatically get an IP address, the default gateway and the DNS server `172.16.1.200`.
- **DNS (my addition):** A DNS server (Server0, `172.16.1.200`) resolves `google.com` to Server1 (`111.0.0.2`) on the ISP side. This lets hosts use names instead of IP addresses.
- **TFTP configuration backup (my addition):** I saved the running configuration of the routers and switches to a TFTP server (`172.16.2.20`) with `copy running-config tftp`, so a configuration can be restored if a device fails or is reset.

## Verification

### DHCP (R3, excerpt)

```
R3#show running-config | section dhcp
ip dhcp excluded-address 172.16.2.225 172.16.2.230
ip dhcp excluded-address 172.16.2.129 172.16.2.134
...
ip dhcp pool VLAN10
 network 172.16.2.224 255.255.255.224
 default-router 172.16.2.225
 dns-server 172.16.1.200
```

### EIGRP neighbors (R3)

```
R3#show ip eigrp neighbors
IP-EIGRP neighbors for process 100
H   Address         Interface      Hold Uptime    SRTT   RTO   Q   Seq
0   200.0.0.5       Se0/0/0        14   01:35:24  40     1000  0   35
1   200.0.0.10      Se0/0/1        14   01:34:43  40     1000  0   36
```

R3 has EIGRP neighbors on both serial links, towards R1 and R4.

### EtherChannel (R3S1)

```
R3S1#show etherchannel summary
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------
1      Po1(SU)           LACP   Fa0/22(P) Fa0/23(P) Fa0/24(P)
3      Po3(SU)           PAgP   Fa0/1(P) Fa0/2(P)
```

Both port-channels are in use and bundled (`SU`, ports in `P` state).

### VTP synchronization

`R3S1` (server) and `R3S3` (client) show the same configuration revision (15) and the same MD5 digest, so VLAN information is synchronized.

```
R3S1#show vtp status
VTP Operating Mode              : Server
VTP Domain Name                 : R3
Configuration Revision          : 15

R3S3#show vtp status
VTP Operating Mode              : Client
VTP Domain Name                 : R3
Configuration Revision          : 15
```

### Configuration backup (TFTP)

```
R3#copy running-config tftp
Address or name of remote host []? 172.16.2.20
Destination filename [R3-confg]? R3-confg
Writing running-config...!!
[OK]
```

### Connectivity tests

- **DNS and internet:** `ping google.com` from a host resolves the name to `111.0.0.2` and gets replies from the ISP-side server. The first packet times out while ARP resolves, then the following replies succeed.

  <img width="1126" height="822" alt="Screenshot 2026-11-25 221207" src="https://github.com/user-attachments/assets/8d635971-892b-4342-b254-71ce9050dbce" />

- **Inter-site routing:** A host at Site 1 pings the DNS server `172.16.1.200` at Site 2 with 0% loss (TTL 126, which means two router hops).

  <img width="1186" height="877" alt="Screenshot 2026-11-25 221307" src="https://github.com/user-attachments/assets/6c2b9efd-3d1f-4fbe-ac59-8a2b0122fc15" />

## What I Learned

- **Plan first, then configure.** A clear IP plan with VLSM made the configuration much easier. Each VLAN got a subnet that fits its number of hosts, and the addresses were easy to check afterwards.
- **Switching needs redundancy and loop protection together.** EtherChannel gives more bandwidth and a backup path. STP stops loops. The place of the root bridge decides which path traffic takes, so it needs to be planned.
- **VTP saves time but needs care.** Creating VLANs once on the server and sharing them is fast. Server, client and transparent modes behave differently, and the revision number must match across switches.
- **Router-on-a-stick connects VLANs with one link.** One physical link with a subinterface per VLAN is enough for inter-VLAN routing.
- **DHCP and DNS work as one chain.** In the test, a host got its address and DNS server from DHCP, resolved `google.com` through DNS, and reached the server through EIGRP and the default route. The first ping timed out while ARP resolved, and then it worked.
- **Small settings improve security.** Port security, passive interfaces, shutting down unused ports and turning off CDP reduce the attack surface. This is the part that made me more interested in network security.
- **Backups protect the work.** Saving each device's running configuration to a TFTP server means a failed or reset device can be restored quickly.
- **`show` commands are the best way to check the work.** I used `show etherchannel summary`, `show vtp status`, `show ip eigrp neighbors` and `ping` to confirm that each part worked before moving on.
