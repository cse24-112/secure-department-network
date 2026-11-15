# Secure Department Network

A multi-VLAN departmental network designed, configured, secured, and
tested in **Cisco Packet Tracer**.

## Project Overview

This project simulates the network infrastructure of a small company
with Management, IT, Finance, and Guest users. The goal was to segment
departments into separate networks, enable controlled communication,
protect internal resources, and provide outbound connectivity through a
simulated ISP.

The network uses a Cisco multilayer switch for inter-VLAN routing,
access switches for endpoint connectivity, a router for upstream routing
and NAT/PAT, and an ISP router with an external test server to simulate
Internet connectivity.

> **Environment:** Cisco Packet Tracer\
> **Project type:** Networking / network security lab\
> **Focus areas:** VLANs, trunking, IPv4, DHCP, Layer 3 switching, ACLs,
> SSH, static routing, NAT/PAT, testing, and troubleshooting.

## Objectives

-   Segment users by department using VLANs.
-   Configure 802.1Q trunk links between switches.
-   Provide inter-VLAN routing using a multilayer switch.
-   Assign client IPv4 settings dynamically using DHCP.
-   Restrict traffic between selected departmental networks.
-   Isolate Guest users from internal networks.
-   Enable secure SSH administration on the core switch.
-   Connect the internal network to a simulated ISP using static routing
    and NAT/PAT.
-   Verify connectivity and security behaviour through testing.

## Topology

The logical network is arranged as follows:

``` text
                         Simulated external network
                                   |
                            INTERNET-SERVER
                               8.8.8.8
                                   |
                                  ISP
                                   |
                               R1 (2911)
                                   |
                             Routed /30 link
                                   |
                         CORE-SW1 (3560)
                         /      |       \
                        /       |        \
                ACCESS-SW1  ACCESS-SW2  ACCESS-SW3
                    |           |          |     \
                Management      IT       Finance  Guest
                  2 PCs        2 PCs      2 PCs   1 PC
```

The core switch provides the departmental VLAN gateways and performs
inter-VLAN routing. R1 provides the upstream path and NAT/PAT. The ISP
router and Internet server represent an external network for lab
testing.

## Devices Used

  ------------------------------------------------------------------------
  Device                                    Quantity Role
  --------------------- ---------------------------- ---------------------
  Cisco 2911 router                                1 Internal edge router,
  (R1)                                               static routing,
                                                     NAT/PAT

  Cisco 3560 multilayer                            1 VLAN gateways and
  switch (CORE-SW1)                                  inter-VLAN routing

  Cisco 2960 access                                3 Connect departmental
  switches                                           end devices

  Cisco 2911 router                                1 Simulated ISP
  (ISP)                                              

  Server                                           1 Simulated external
  (INTERNET-SERVER)                                  destination

  PCs                                              7 Management, IT,
                                                     Finance, and Guest
                                                     endpoints
  ------------------------------------------------------------------------

**Endpoint breakdown:** 2 Management PCs, 2 IT PCs, 2 Finance PCs, and 1
Guest PC.

## VLAN and IPv4 Addressing Plan

    VLAN ID Name                 Subnet              Default gateway
  --------- -------------------- ------------------- -----------------
         10 MANAGEMENT           `192.168.10.0/24`   `192.168.10.1`
         20 IT                   `192.168.20.0/24`   `192.168.20.1`
         30 FINANCE              `192.168.30.0/24`   `192.168.30.1`
         40 GUEST                `192.168.40.0/24`   `192.168.40.1`
         99 NETWORK-MANAGEMENT   `192.168.99.0/24`   `192.168.99.1`

The first ten addresses in the four client subnets were excluded from
DHCP allocation to reserve them for infrastructure or static
assignments.

### Routed link and simulated WAN

  Link / device     Interface              IPv4 address
  ----------------- ---------------------- ------------------
  CORE-SW1          Routed uplink to R1    `10.0.0.2/30`
  R1                `GigabitEthernet0/0`   `10.0.0.1/30`
  R1                `GigabitEthernet0/1`   `203.0.113.2/30`
  ISP               `GigabitEthernet0/0`   `203.0.113.1/30`
  ISP               `GigabitEthernet0/1`   `8.8.8.1/24`
  INTERNET-SERVER   Network interface      `8.8.8.8/24`
  INTERNET-SERVER   Default gateway        `8.8.8.1`

The `203.0.113.0/24` range is reserved for documentation, and
`8.8.8.0/24` is used here as a lab-only simulated external subnet. The
server address `8.8.8.8` in this topology is **not** Google's public DNS
service.

## Switching Configuration

VLANs 10, 20, 30, 40, and 99 were created on the core switch and the
relevant VLANs were created on the access switches.

Departmental endpoint ports were configured as access ports: -
ACCESS-SW1: Management endpoints on `Fa0/2` and `Fa0/3`, VLAN 10. -
ACCESS-SW2: IT endpoints on `Fa0/2` and `Fa0/3`, VLAN 20. - ACCESS-SW3:
Finance endpoints on `Fa0/2` and `Fa0/3`, VLAN 30. - ACCESS-SW3: Guest
endpoint on `Fa0/4`, VLAN 40.

The access-switch uplinks use `Fa0/1`. The links from the access
switches to CORE-SW1 were configured as 802.1Q trunks, carrying the
required VLANs.

The core switch's link to R1 was configured as a routed Layer 3 port
rather than a VLAN trunk.

## Inter-VLAN Routing

The Cisco 3560 multilayer switch performs inter-VLAN routing. Switched
Virtual Interfaces (SVIs) were configured as the default gateways for
the five VLANs, and Layer 3 routing was enabled with:

``` text
ip routing
```

A routed point-to-point link connects CORE-SW1 (`10.0.0.2/30`) to R1
(`10.0.0.1/30`). A default route on CORE-SW1 points upstream to R1.

R1 has static routes pointing the internal VLAN subnets back to
CORE-SW1.

## DHCP

DHCP was configured on CORE-SW1 for the client VLANs:

-   MANAGEMENT --- `192.168.10.0/24`
-   IT --- `192.168.20.0/24`
-   FINANCE --- `192.168.30.0/24`
-   GUEST --- `192.168.40.0/24`

The DHCP pools provide each client with an address in its subnet and the
corresponding VLAN gateway as its default router. Client PCs were set to
obtain their IPv4 configuration automatically.

VLAN 99 has an SVI and subnet in the addressing plan, but a DHCP pool
for VLAN 99 was not part of the configured client DHCP pools.

## Security Controls

### Guest isolation

An extended ACL named `GUEST_RESTRICTION` was applied inbound on SVI
`Vlan40`. It denies Guest-originated IP traffic to the internal subnets:

-   `192.168.10.0/24` --- Management
-   `192.168.20.0/24` --- IT
-   `192.168.30.0/24` --- Finance
-   `192.168.99.0/24` --- Network Management

It then permits other IP destinations. This allows Guest traffic to
reach destinations outside the listed internal subnets, subject to the
rest of the network configuration.

### Finance restrictions

An extended ACL named `FINANCE_RESTRICTION` was applied inbound on SVI
`Vlan30`. It denies Finance-originated traffic to:

-   `192.168.20.0/24` --- IT
-   `192.168.40.0/24` --- Guest

It permits traffic from Finance to Management and then permits other IP
destinations. Therefore, this ACL does **not** restrict Finance to
Management only; other destinations not explicitly denied are also
permitted.

### SSH administration

SSH was configured on CORE-SW1 using a local administrative account, a
domain name, RSA keys, local VTY authentication, and SSH-only transport
on the VTY lines. Telnet was not enabled on those VTY lines.

The lab credentials are intentionally not included in this repository.
Use your own credentials if recreating the lab, and never publish
passwords or private keys.

> **Security scope:** The ACLs described above are the ACLs configured
> in this lab. They are not a complete enterprise firewall policy. In
> particular, the Finance ACL permits destinations other than the two
> explicitly denied subnets. VLAN 99 was created, but dedicated
> management addressing on each access switch and a full
> management-plane access policy were not completed as part of the
> recorded configuration.

## NAT/PAT and Simulated Internet Access

R1 was configured with: - An inside interface toward CORE-SW1. - An
outside interface toward the ISP. - A standard ACL matching internal
`192.168.0.0/16` source addresses. - NAT overload (PAT) using the WAN
interface address. - A default route toward the ISP at `203.0.113.1`.

The ISP router provides the simulated external subnet `8.8.8.0/24`,
where the Internet server is configured as `8.8.8.8/24` with gateway
`8.8.8.1`.

This is a **Packet Tracer simulation of external connectivity**, not a
connection to the public Internet.

## Verification and Testing

The following checks were used during the build:

  -----------------------------------------------------------------------
  Test                                Expected / observed purpose
  ----------------------------------- -----------------------------------
  PC to its own VLAN gateway          Confirm local VLAN connectivity

  DHCP on Management, IT, Finance,    Confirm automatic addressing
  and Guest PCs                       

  `show vlan brief`                   Inspect VLAN creation and port
                                      membership

  `show interfaces trunk`             Verify trunk operation

  `show ip interface brief`           Check interface and SVI status

  `show ip route`                     Inspect connected and static routes

  `show ip dhcp binding`              Inspect active DHCP leases

  `show access-lists`                 Inspect configured ACL entries and
                                      counters

  `show ip ssh`                       Verify SSH status on CORE-SW1

  CORE-SW1 to R1 ping                 Verify the routed core-to-router
                                      link

  R1 to ISP ping                      Verify the simulated WAN link

  Client to `8.8.8.8` ping            Verify the end-to-end simulated
                                      external path

  Guest to Finance gateway ping       Compare baseline reachability with
  before and after ACL                Guest isolation
  -----------------------------------------------------------------------

The Guest-to-Finance test succeeded before the ACL was applied and was
expected to fail after the ACL was applied. Finance-to-IT was also
tested against the Finance restriction. Successful pings to permitted
destinations and failed pings to denied destinations were used to
validate policy behaviour.

## Troubleshooting and Lessons Learned

During implementation, the following issues were encountered and
resolved:

1.  **Trunk encapsulation on different switch models:** The 3560
    required `switchport trunk encapsulation dot1q` before trunk mode
    could be enabled. The 2960 did not accept that encapsulation command
    because it uses 802.1Q.
2.  **DHCP pool configuration:** An incorrect network address and a
    pool-name typo affected the IT pool. The incorrect pool was removed,
    the IT pool was recreated with `192.168.20.0/24`, and IT clients
    subsequently obtained addresses.
3.  **Routed core-to-router connection:** A dedicated point-to-point
    `/30` network was used between the multilayer switch and R1, with
    static routes in both directions.
4.  **Security policy validation:** Connectivity was tested before and
    after applying ACLs to observe the difference between an
    unrestricted baseline and the intended restrictions.

The project reinforced the importance of checking interface status, VLAN
membership, trunk state, DHCP leases, routing tables, and ACL behaviour
rather than assuming a configuration is correct.

## Skills Demonstrated

-   Cisco Packet Tracer network design
-   VLAN creation and access-port assignment
-   802.1Q trunking
-   IPv4 subnetting and addressing
-   Layer 3 switching and inter-VLAN routing
-   DHCP configuration and troubleshooting
-   Extended ACL configuration and verification
-   SSH device administration
-   Static routing
-   NAT/PAT (overload)
-   Connectivity testing with `ping`
-   Network troubleshooting and documentation

## Repository Contents

Suggested repository structure:

``` text
secure-department-network/
├── README.md
├── topology/
│   └── secure-department-network.png
├── packet-tracer/
│   └── secure-department-network.pkt
└── screenshots/
    ├── 01-topology.png
    ├── 02-vlans.png
    ├── 03-routing-table.png
    ├── 04-acls.png
    ├── 05-dhcp-bindings.png
    ├── 06-ssh-status.png
    ├── 07-internet-connectivity.png
    └── 08-guest-isolation.png
```

Add the Packet Tracer `.pkt` file and screenshots only after checking
that they are the final versions and contain no real credentials or
personal information.

## Future Improvements

Possible next steps to extend the lab:

-   Assign and verify management IP addresses on the access switches.
-   Restrict management access to trusted administrator devices.
-   Review and refine the Finance ACL to explicitly define every
    permitted destination.
-   Apply consistent SSH and hardening settings to all network devices.
-   Add a dedicated test matrix for every permitted and denied traffic
    flow.
-   Add redundant links or devices and test failover.
-   Export and document the final device configurations.

## Disclaimer

This is an educational lab built in Cisco Packet Tracer. Device
behaviour, addressing, and external connectivity are simulated. The
design is intended to demonstrate networking concepts and is not
presented as a production-ready enterprise deployment.
