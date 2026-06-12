# 🌐 Redistribution Mega Lab 2

## 📌 Overview
Large-scale network lab implementing route redistribution
between three routing protocols: OSPF, EIGRP, and RIPv2.
Goal: Full connectivity from PC-1 (R21) to PC-4 (R39)
across all routing domains.

---

## 🗺️ Network Topology

### Routing Protocols Used
| Protocol | Area/AS | Routers |
|----------|---------|---------|
| OSPF 10 | Area 1 | R1 to R5 |
| OSPF 10 | Area 0 (Backbone) | R5 to R10 |
| OSPF 10 | Area 2 | R5 and R11 to R17 |
| OSPF 10 | Area 3 | R15 and R18 to R23 |
| EIGRP 100 | AS 100 | R23 to R28 |
| EIGRP 50 | AS 50 | R31 and R34 to R40 |
| RIPv2 | — | R28 to R34 |

---

## 📋 IP Addressing Scheme

### WAN Links
- Network: 1.0.0.0/8 to 40.0.0.0/8
- Each router link gets its own network ID

### LAN (Gateway) Networks
- Range: 192.168.1.0/24 to 192.168.40.0/24
- Each PC subnet gets its own gateway

### Key Networks
| Location | Network |
|----------|---------|
| LAN | 192.168.1.0/24 |
| WAN | 1.0.0.0/8 |
| Static (R38) | 172.16.x.x/25 range |

---

## 📌 Why I Gave Gateway to Both PCs and Routers

### Gateway on PCs
- PCs are end devices and do not know
  where to send traffic outside their subnet
- They need a gateway to forward all
  unknown traffic to their connected router
- Example: PC-1 gateway = 192.168.1.1 (R21)

### Gateway on Routers
- Routers also received a default gateway
  in this lab
- Reason: Routing protocols (OSPF, EIGRP, RIPv2)
  only learn routes that are configured under
  their network statements
- Any traffic that does not match a known
  route needs a default gateway to be forwarded
- Without a default gateway on the router
  unknown traffic would be dropped silently
- The gateway on router acts as a last resort
  path for unmatched traffic

    ip route 0.0.0.0 0.0.0.0 [next-hop-ip]

### Summary
| Device | Why Gateway Needed |
|--------|--------------------|
| PC | Does not know any routes at all |
| Router | Needs last resort path for unknown traffic |

---

## 📌 Why No Keepalive Only on Ethernet Not Serial

### Ethernet Interfaces
- In EVE-NG lab environment ethernet interfaces
  do not have a real physical carrier signal
- Without no keepalive the interface checks
  for a physical signal and goes down when
  it does not find one
- No keepalive tells the interface to stay up
  even without a physical signal

    interface ethernet 0/0
    no keepalive

### Serial Interfaces
- Serial interfaces in EVE-NG work differently
- They use their own clock rate and timing
  mechanism to stay up
- They do not rely on physical carrier signal
  the same way ethernet does
- So no keepalive is NOT needed on serial

### Summary
| Interface | No Keepalive Needed |
|-----------|---------------------|
| Ethernet | YES — no physical signal in EVE-NG |
| Serial | NO — uses clock rate mechanism |

---

## ✅ What I Configured

### 1. IP Assignment on Routers
- Assigned WAN IPs to all serial interfaces
- Assigned LAN IPs as gateways for each PC subnet
- Configured 192.168.1.0/24 to 192.168.40.0/24
  as gateway networks for all PCs
- Gave default gateway to routers for
  unknown traffic forwarding

    enable
    configure terminal
    interface serial 0/0
    ip address 1.0.0.1 255.0.0.0
    no shutdown
    exit
    interface ethernet 0/0
    ip address 192.168.1.1 255.255.255.0
    no shutdown
    no keepalive
    exit
    ip route 0.0.0.0 0.0.0.0 [next-hop-ip]

### 2. OSPF 10 Configuration (Areas 0,1,2,3)

    router ospf 10
    network 1.0.0.0 0.255.255.255 area 1
    network 2.0.0.0 0.255.255.255 area 1
    network 3.0.0.0 0.255.255.255 area 1
    network 4.0.0.0 0.255.255.255 area 1
    network 5.0.0.0 0.255.255.255 area 0
    network 6.0.0.0 0.255.255.255 area 0
    network 7.0.0.0 0.255.255.255 area 0
    network 8.0.0.0 0.255.255.255 area 0
    network 9.0.0.0 0.255.255.255 area 0
    network 10.0.0.0 0.255.255.255 area 0
    network 11.0.0.0 0.255.255.255 area 2
    network 12.0.0.0 0.255.255.255 area 2
    network 13.0.0.0 0.255.255.255 area 2
    network 14.0.0.0 0.255.255.255 area 2
    network 15.0.0.0 0.255.255.255 area 2
    network 16.0.0.0 0.255.255.255 area 2
    network 17.0.0.0 0.255.255.255 area 2
    network 18.0.0.0 0.255.255.255 area 3
    network 19.0.0.0 0.255.255.255 area 3
    network 20.0.0.0 0.255.255.255 area 3
    network 21.0.0.0 0.255.255.255 area 3
    network 22.0.0.0 0.255.255.255 area 3
    network 23.0.0.0 0.255.255.255 area 3

### 3. EIGRP 100 Configuration

    router eigrp 100
    network 24.0.0.0
    network 25.0.0.0
    network 26.0.0.0
    network 27.0.0.0
    network 28.0.0.0
    no auto-summary

### 4. EIGRP 50 Configuration

    router eigrp 50
    network 31.0.0.0
    network 35.0.0.0
    network 36.0.0.0
    network 37.0.0.0
    network 38.0.0.0
    network 39.0.0.0
    network 40.0.0.0

    no auto-summary

### 5. RIPv2 Configuration

    router rip
    version 2
    network 28.0.0.0
    network 29.0.0.0
    network 30.0.0.0
    network 31.0.0.0
    network 32.0.0.0
    network 33.0.0.0
    network 34.0.0.0
    no auto-summary

### 6. Static Route on R38
- Configured static route on R38
- Reason: R38 connects to external LAN network
- Static route ensures stable path to
  172.16.x.x networks

    ip route 172.16.18.0 255.255.255.128 [next-hop-ip]
    ip route 172.16.19.0 255.255.255.0 [next-hop-ip]
    ip route 172.16.20.0 255.255.255.0 [next-hop-ip]
    ip route 172.16.21.0 255.255.255.0 [next-hop-ip]
    ip route 172.16.22.0 255.255.255.0 [next-hop-ip]
    ip route 172.16.23.0 255.255.255.0 [next-hop-ip]
    ip route 172.16.24.0 255.255.255.0 [next-hop-ip]
    ip route 172.16.25.0 255.255.255.0 [next-hop-ip]

### 7. Route Redistribution

#### R23 — OSPF 10 and EIGRP 100

    router ospf 10
    redistribute eigrp 100 subnets

    router eigrp 100
    redistribute ospf 10 metric 10000 100 255 1 1500

#### R28 — EIGRP 100 and RIPv2

    router eigrp 100
    redistribute rip metric 10000 100 255 1 1500

    router rip
    version 2
    redistribute eigrp 100 metric 5

#### R31 — EIGRP 50 and RIPv2

    router eigrp 50
    redistribute rip metric 10000 100 255 1 1500

    router rip
    version 2
    redistribute eigrp 50 metric 5

### 8. Verification Commands

    show ip route
    show ip ospf neighbor
    show ip eigrp neighbors
    show ip rip database
    ping 192.168.40.1 source 192.168.1.1

### 9. End to End Ping Test
- PC-1 (connected to R21) pinged PC-4 (connected to R39)
- Path crosses OSPF10-A3 → RIPv2 → EIGRP50
- Ping successful ✅

---

## 🔄 Redistribution Flow

    [PC-1]
      |
    [R21] — EIGRP 100
               |
              [R23] — Redistribution OSPF10 and EIGRP100
               |
              [R28] — Redistribution EIGRP100 and RIPv2
               |
              [R31] — Redistribution EIGRP50 and RIPv2
               |
              [R39]
                |
              [PC-4]

---

## ❌ Errors I Faced & How I Fixed Them

### Error 1 — Routes Not Appearing in Routing Table
**Problem:**
Routes from OSPF not visible in EIGRP routers

**Cause:**
Missing subnets keyword in redistribution command

**Fix:**

    router ospf 10
    redistribute eigrp 100 subnets

Adding subnets keyword fixed it ✅

---

### Error 2 — Ping Failing from PC1 to PC4
**Problem:**
Ping was failing even after redistribution

**Cause:**
Gateway IP was missing on PC side

**Fix:**
- Added correct gateway IP on PC-1 and PC-4
- Ping worked after adding gateway ✅

---

### Error 3 — Ethernet Interface Going Down
**Problem:**
Ethernet interfaces kept going down in EVE-NG lab

**Cause:**
No real physical signal on ethernet interfaces
in EVE-NG environment

**Fix:**

    interface ethernet 0/0
    no keepalive

Interface stayed up and stable ✅

---

### Error 4 — EIGRP Routes Missing After Redistribution
**Problem:**
EIGRP routes not showing after redistributing RIP

**Cause:**
Auto-summary was enabled causing route summarization

**Fix:**

    router eigrp 100
    no auto-summary

All routes appeared correctly ✅

---

### Error 5 — RIPv2 Not Redistributing Correctly
**Problem:**
RIP routes not passing to EIGRP properly

**Cause:**
Metric was not defined during redistribution

**Fix:**

    router eigrp 100
    redistribute rip metric 10000 100 255 1 1500

Metric values added and routes appeared ✅

---

## 🔧 Tools Used
- EVE-NG — used to build and run the full
  network topology with real Cisco router images
- SecureCRT — used to connect to each router
  via terminal and enter all commands
- OSPF, EIGRP, RIPv2 routing protocols
- Static routing
- Route redistribution

---

## 📚 What I Learned
- How to redistribute between different protocols
- Why subnets keyword is needed in OSPF
- Why no keepalive is only needed on ethernet
  interfaces and not serial interfaces
- Why both routers and PCs need a gateway
  but for different reasons
- How default gateway on router handles
  unknown traffic as last resort
- How to verify routes using show ip route
- How packets travel across different
  routing protocol domains
- How static routes work on border routers
- Why no auto-summary is important in EIGRP
- How metric values affect redistribution

---

## 🎯 Tasks Completed
- [x] Configure IPs and Protocols on all routers
- [x] Apply no keepalive on ethernet interfaces only
- [x] Configure default gateway on all routers
- [x] Configure gateway on all PC subnets
- [x] Configure OSPF 10 Areas 0,1,2,3
- [x] Configure EIGRP 100 and EIGRP 50
- [x] Configure RIPv2
- [x] Redistribute on R23, R28 and R31
- [x] Configure static routes on R38
- [x] Verify routing tables on all routers
- [x] Test ping from PC-1 to PC-4 successfully

---

## 📌 Why Metric is Necessary in Redistribution

### The Problem Without Metric
- When you redistribute routes from one protocol
  to another the receiving protocol does not know
  what cost to assign to that route
- EIGRP uses its own metric system
- RIP uses hop count
- OSPF uses cost
- Each protocol speaks a different language
  for measuring path cost
- Without metric the router cannot install
  the redistributed route into routing table
- The route would be ignored or dropped

### EIGRP Metric — What the Numbers Mean

    redistribute ospf 10 metric 10000 100 255 1 1500
    redistribute rip metric 10000 100 255 1 1500

| Position | Value | Meaning |
|----------|-------|---------|
| 1st | 10000 | Bandwidth in Kbps |
| 2nd | 100 | Delay in microseconds |
| 3rd | 255 | Reliability (255 = 100% reliable) |
| 4th | 1 | Load (1 = minimum load) |
| 5th | 1500 | MTU (Maximum Transmission Unit) |

- Bandwidth 10000 = 10 Mbps link speed
- Delay 100 = how long it takes to traverse link
- Reliability 255 = most reliable (scale 0 to 255)
- Load 1 = least loaded (scale 1 to 255)
- MTU 1500 = standard ethernet frame size
- EIGRP uses bandwidth and delay most heavily
  to calculate its composite metric

### RIP Metric — What the Number Means

    redistribute eigrp 100 metric 5
    redistribute eigrp 50 metric 5

- RIP only uses hop count as metric
- metric 5 means this route is 5 hops away
- RIP maximum hop count is 15
- Anything above 15 is unreachable
- We use 5 so there is room for more hops
  if the route passes through more RIP routers
- If you set metric 1 it looks like directly
  connected which may cause routing loops

### OSPF Metric — subnets Keyword

    redistribute eigrp 100 subnets

- By default OSPF only redistributes
  classful networks (Class A B C)
- Without subnets keyword OSPF ignores
  all subnetted routes like 192.168.1.0/24
- The subnets keyword tells OSPF to also
  redistribute classless and subnetted routes
- Without it routes simply disappear
  and do not appear in routing table
- Always add subnets when redistributing
  into OSPF

### Summary Table
| Protocol | Metric Keyword | What It Means |
|----------|---------------|---------------|
| EIGRP | metric 10000 100 255 1 1500 | BW Delay Reliability Load MTU |
| RIP | metric 5 | Hop count = 5 |
| OSPF | subnets | Include subnetted routes |

---

## 📋 Corrected Routing Protocol Zones

| Protocol | Area/AS | Routers |
|----------|---------|---------|
| OSPF 10 | Area 1 | R1 to R5 |
| OSPF 10 | Area 0 (Backbone) | R5 to R10 and R18 |
| OSPF 10 | Area 2 | R5 and R11 to R17 |
| OSPF 10 | Area 3 | R15 and R18 to R23 |
| EIGRP 100 | AS 100 | R23 to R28 |
| RIPv2 | — | R28 to R34 |
| EIGRP 50 | AS 50 | R31 and R34 to R40 |

---

## 🔄 Corrected Redistribution Flow

    [PC-1]
      |
    [R21] — OSPF Area 3
               |
              [R23] — Redistribution OSPF Area3 and EIGRP100
               |
           EIGRP 100 (R23 to R28)
               |
              [R28] — Redistribution EIGRP100 and RIPv2
               |
           RIPv2 (R28 to R34)
               |
              [R31] — Redistribution RIPv2 and EIGRP50
               |
           EIGRP 50 (R34 to R40)
               |
              [R39]
                |
              [PC-4]


