# TechCore Solutions Enterprise Network

## Project Overview

This project demonstrates the design and configuration of a small
enterprise network in **Cisco Packet Tracer**. The simulated company,
TechCore Solutions, has separate Administration, IT, Sales, and Server
networks.

The network uses VLANs to separate departments, a router to route
traffic between VLANs, DHCP to automatically assign IP addresses to the
PCs, and an extended Access Control List (ACL) to restrict Sales from
accessing the server network.

> **Project status:** Update this section after you have built and
> tested the Packet Tracer simulation. Only describe tests as successful
> if you have run them and confirmed the results.

## Objectives

-   Design a small enterprise network topology.
-   Create and configure four VLANs.
-   Configure switch access ports and an 802.1Q trunk.
-   Configure router-on-a-stick inter-VLAN routing.
-   Configure DHCP pools on the router for the three PCs.
-   Give the server a static IPv4 address.
-   Use an extended ACL to block Sales access to the server subnet.
-   Test connectivity and document the results.

## Network Topology

The topology contains:

-   1 Cisco 2911 router (`R1`)
-   1 Cisco 2960 switch (`S1`)
-   3 PCs (`ADMIN-PC`, `IT-PC`, `SALES-PC`)
-   1 server (`SERVER1`)

### Cable Connections

Use copper straight-through cables for these connections:

  Device A   Port A                 Device B   Port B
  ---------- ---------------------- ---------- ----------------------
  R1         `GigabitEthernet0/0`   S1         `GigabitEthernet0/1`
  ADMIN-PC   `FastEthernet0`        S1         `FastEthernet0/2`
  IT-PC      `FastEthernet0`        S1         `FastEthernet0/3`
  SALES-PC   `FastEthernet0`        S1         `FastEthernet0/4`
  SERVER1    `FastEthernet0`        S1         `FastEthernet0/5`

The router-to-switch link carries traffic for all four VLANs. The switch
port connected to the router is configured as a trunk.

## VLAN and IP Addressing Plan

  Department / Network     VLAN ID Network             Default Gateway
  ---------------------- --------- ------------------- -----------------
  Administration                10 `192.168.10.0/24`   `192.168.10.1`
  IT                            20 `192.168.20.0/24`   `192.168.20.1`
  Sales                         30 `192.168.30.0/24`   `192.168.30.1`
  Servers                       40 `192.168.40.0/24`   `192.168.40.1`

The `/24` prefix corresponds to subnet mask `255.255.255.0`.

### End-Device Addressing

  -----------------------------------------------------------------------
  Device            Addressing Method IP Address /      Default Gateway
                                      Expected Range    
  ----------------- ----------------- ----------------- -----------------
  ADMIN-PC          DHCP              `192.168.10.x`    `192.168.10.1`

  IT-PC             DHCP              `192.168.20.x`    `192.168.20.1`

  SALES-PC          DHCP              `192.168.30.x`    `192.168.30.1`

  SERVER1           Static            `192.168.40.10`   `192.168.40.1`
  -----------------------------------------------------------------------

The router provides DHCP for the three PCs. The server uses a static
address so its address remains predictable.

## Configuration Summary

### 1. Switch S1

The switch has four VLANs:

    VLAN Name        Switch Access Port
  ------ ----------- --------------------
      10 `ADMIN`     `FastEthernet0/2`
      20 `IT`        `FastEthernet0/3`
      30 `SALES`     `FastEthernet0/4`
      40 `SERVERS`   `FastEthernet0/5`

The `GigabitEthernet0/1` port connected to R1 is configured as an 802.1Q
trunk and allows VLANs 10, 20, 30, and 40.

Useful verification commands on S1:

``` text
show vlan brief
show interfaces trunk
show interfaces status
```

### 2. Router R1: Inter-VLAN Routing

R1 uses router-on-a-stick. Its physical `GigabitEthernet0/0` interface
connects to S1, while four subinterfaces provide the VLAN gateways.

  Router Subinterface         VLAN IP Address
  ------------------------- ------ -------------------
  `GigabitEthernet0/0.10`       10 `192.168.10.1/24`
  `GigabitEthernet0/0.20`       20 `192.168.20.1/24`
  `GigabitEthernet0/0.30`       30 `192.168.30.1/24`
  `GigabitEthernet0/0.40`       40 `192.168.40.1/24`

Each subinterface uses `encapsulation dot1Q` with its matching VLAN ID.
The physical interface must be enabled with `no shutdown`.

Useful verification commands on R1:

``` text
show ip interface brief
show ip route
```

### 3. DHCP on R1

R1 provides DHCP pools for:

-   `ADMIN-POOL` --- network `192.168.10.0/24`, gateway `192.168.10.1`
-   `IT-POOL` --- network `192.168.20.0/24`, gateway `192.168.20.1`
-   `SALES-POOL` --- network `192.168.30.0/24`, gateway `192.168.30.1`

Addresses `192.168.x.1` through `192.168.x.20` are excluded from DHCP
for these PC subnets to reserve them for gateways and other
infrastructure. Each PC is set to **Desktop → IP Configuration → DHCP**.

Verification commands:

``` text
show ip dhcp binding
show ip dhcp pool
```

### 4. Server Configuration

`SERVER1` uses the following static IPv4 configuration:

-   IP address: `192.168.40.10`
-   Subnet mask: `255.255.255.0`
-   Default gateway: `192.168.40.1`

HTTP can optionally be enabled under the server's **Services → HTTP**
tab to simulate an internal company website. If enabled, test it from a
browser using `http://192.168.40.10`.

### 5. ACL Security Policy on R1

The extended ACL is named `SALES-RESTRICTIONS`. It denies IP traffic
from the Sales subnet (`192.168.30.0/24`) to the server subnet
(`192.168.40.0/24`) and permits other IP traffic that is not matched by
the deny rule.

``` text
ip access-list extended SALES-RESTRICTIONS
 deny ip 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip any any
```

Apply the ACL **inbound** on router subinterface
`GigabitEthernet0/0.30`:

``` text
interface gigabitEthernet0/0.30
 ip access-group SALES-RESTRICTIONS in
```

The ACL is placed on the Sales subinterface because Sales traffic enters
the router through that interface. The `permit ip any any` line allows
other traffic not matched by the deny rule; without it, the ACL's
implicit deny would block all remaining traffic on that interface.

This is a basic lab segmentation policy, not a complete enterprise
firewall or comprehensive security design.

Verification commands:

``` text
show access-lists
show ip interface gigabitEthernet0/0.30
```

## Testing Plan

Run these tests after completing the configuration. Record the actual
results in the table rather than assuming they passed.

  ------------------------------------------------------------------------
  Test                    Where to Run It          Expected Result
  ----------------------- ------------------------ -----------------------
  Check IP settings       Each PC: `ipconfig`      PC has an address in
                                                   its assigned subnet and
                                                   the correct gateway

  Ping Administration     ADMIN-PC:                Success
  gateway                 `ping 192.168.10.1`      

  Ping IT gateway         IT-PC:                   Success
                          `ping 192.168.20.1`      

  Ping Sales gateway      SALES-PC:                Success
                          `ping 192.168.30.1`      

  Ping server from Admin  ADMIN-PC:                Success
                          `ping 192.168.40.10`     

  Ping server from IT     IT-PC:                   Success
                          `ping 192.168.40.10`     

  Ping server from Sales  SALES-PC:                Blocked by the ACL
                          `ping 192.168.40.10`     

  Check DHCP leases       R1:                      Leases are shown for
                          `show ip dhcp binding`   PCs that obtained
                                                   addresses

  Check ACL counters      R1: `show access-lists`  Deny counter increases
                                                   after blocked Sales
                                                   traffic is attempted

  Test HTTP, if enabled   Admin/IT browser:        Page loads for allowed
                          `http://192.168.40.10`   departments; Sales
                                                   access is blocked
  ------------------------------------------------------------------------

A first ping may time out while ARP resolves addresses. Repeat the test
before treating a single timeout as a failure.

## Troubleshooting Notes

-   **PC has a `169.254.x.x` address:** Check DHCP pools, VLAN
    assignment, trunk configuration, and router subinterfaces.
-   **PC cannot ping its gateway:** Check the PC's IP settings and its
    switch access VLAN.
-   **All departments cannot reach the server:** Check the server's
    static IP, subnet mask, gateway, VLAN 40 port, and router
    subinterface.
-   **Sales can reach the server:** Confirm the ACL is applied inbound
    on `GigabitEthernet0/0.30` and that the deny entry appears before
    the permit entry.
-   **Trunk is not active:** Run `show interfaces trunk` on S1 and
    confirm the router-connected switch port is trunking.

## Skills Demonstrated

-   Cisco Packet Tracer network design
-   VLAN creation and segmentation
-   Switch access-port configuration
-   802.1Q trunking
-   Router-on-a-stick inter-VLAN routing
-   IPv4 subnetting and default gateways
-   DHCP configuration and verification
-   Extended ACL configuration
-   Basic network security policy enforcement
-   Connectivity testing and troubleshooting
-   Technical documentation

## Project Files

Recommended repository layout:

``` text
TechCore-Enterprise-Network/
├── README.md
├── TechCore_Enterprise_Network.pkt
├── configuration-notes.md
└── screenshots/
    ├── topology.png
    ├── vlan-configuration.png
    ├── trunk-verification.png
    ├── dhcp-leases.png
    ├── allowed-server-access.png
    └── sales-access-blocked.png
```

Add the Packet Tracer `.pkt` file and screenshots after creating them.
Do not include screenshots or files that you have not created yet.

## What I Learned

This project is designed to practise how VLANs separate departments, how
a router enables communication between different IP networks, how DHCP
reduces manual IP configuration, and how an ACL can enforce a simple
access policy. Completing the tests also provides practice in checking
configurations and diagnosing common connectivity issues.

## CV Description

**TechCore Solutions Enterprise Network --- Cisco Packet Tracer**

Designed and simulated an enterprise network with four VLANs for
Administration, IT, Sales, and Servers. Configured 802.1Q trunking,
router-on-a-stick inter-VLAN routing, DHCP address pools, static server
addressing, and an extended ACL to restrict Sales access to the server
subnet. Tested network connectivity, DHCP assignments, and
access-control behavior in Cisco Packet Tracer.

*Use this CV description after you have implemented and tested the
project, and describe only the results you personally verified.*
