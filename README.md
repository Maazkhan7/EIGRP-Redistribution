<div align="center">

# 🔀 EIGRP Redistribution

### RIPv2 + Static Routes → EIGRP · Seed Metrics · External Routes (D EX) · Route Priority

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-EIGRP%20AS%201-blue?style=for-the-badge)
![RIP](https://img.shields.io/badge/Source-RIPv2-green?style=for-the-badge)
![Static](https://img.shields.io/badge/Source-Static%20Routes-purple?style=for-the-badge)
![AD](https://img.shields.io/badge/External%20AD-170-orange?style=for-the-badge)
![Routers](https://img.shields.io/badge/Routers-3%20×%201941-yellow?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%2F%20CCNP-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

**Three routing sources · one EIGRP domain · full control of the metric**

</div>

---

## 📖 Table of Contents

[Overview](#-project-overview) · [Lab Snapshot](#-lab-snapshot) · [Timeline](#-what-happens-in-this-lab-timeline) · [Topology](#️-network-topology) · [IP Table](#-ip-addressing-table) · [Configuration](#️-configuration) · [Stage-by-Stage Verification](#-verification--stage-by-stage) · [Metric Math](#-metric-calculations) · [Command Breakdown](#-redistribute-command-breakdown) · [Troubleshooting](#-troubleshooting-guide) · [Advanced](#-advanced-topics) · [Takeaways](#-key-takeaways) · [Challenges](#-challenges)

---

## 📌 Project Overview

Real networks rarely run one routing protocol. This lab connects **three different route sources** into one **EIGRP AS 1** domain using **route redistribution**:

| Source | Where | What it provides |
|:------:|:-----:|:-----------------|
| 🟡 **EIGRP AS 1** | Router0 · Router1 · Router2 | The main routing domain + loopbacks |
| 🟣 **RIPv2** | Router2's LAN side | `192.168.2.0/24` |
| 🌸 **Static routes** | Router2 | `100.100.100.0/24` · `100.100.150.0/24` · `100.100.200.0/24` → `Null0` |

**Router2 is the ASBR** (Autonomous System Boundary Router): it injects the RIP and static routes into EIGRP, so **Router0 and Router1 can reach them**.

The lab goes further than just turning redistribution on. It shows how the **seed metric** decides how a redistributed route looks and ranks in the routing table:

> 🎯 **Result:** redistributed routes appear as **`D EX`** (EIGRP external, **AD 170**), and by changing `metric 1 1 1 1 1` to `metric 5 5 5 5 5` the **RIP route becomes far more preferred** than the static routes.

---

## 📸 Lab Snapshot

| Item | Value |
|:-----|:------|
| 🧰 Tool | Cisco Packet Tracer |
| 🖧 Routers | 3 × Cisco **1941** (Router0, Router1, Router2) |
| 🔀 Switches / PCs | 2 × 2950-24 / 6 × PC-PT |
| 🌐 Main domain | **EIGRP AS 1** |
| 🔁 Redistributed into EIGRP | **RIP** and **static** (on the ASBR, Router2) |
| 🧭 External route code | **`D EX`** · AD **170** |
| 🔢 Seed metrics tested | `none` · `1 1 1 1 1` · `5 5 5 5 5` |
| 🏆 Key results | `25856` → `2560000512` → `512001536` |

---

## 🎬 What Happens in This Lab (Timeline)

```
 STAGE ①                          STAGE ②                          STAGE ③
 redistribute rip metric 1 1 1 1 1   redistribute static metric 1 1 1 1 1   redistribute rip metric 5 5 5 5 5
 static: NO metric                   static now has a metric             RIP gets a LOWER metric
 ───────────────────────            ───────────────────────            ───────────────────────
 static  → 25856                     static  → 2,560,000,512             static  → 2,560,000,512
 RIP     → 2,560,000,512             RIP     → 2,560,000,512             RIP     → 512,001,536  ⭐ preferred
```

| Stage | Command on Router2 | Static routes (100.100.x.0) | RIP route (192.168.2.0) |
|:-----:|:-------------------|:---------------------------:|:-----------------------:|
| ① | `redistribute rip metric 1 1 1 1 1` *(static without metric)* | `[170/25856]` | `[170/2560000512]` |
| ② | `redistribute static metric 1 1 1 1 1` | `[170/2560000512]` | `[170/2560000512]` — **all equal** |
| ③ | `redistribute rip metric 5 5 5 5 5` | `[170/2560000512]` | **`[170/512001536]`** — lower = better |

---

## 🎯 Objectives

- ✅ Run **EIGRP**, **RIPv2** and **static routes** in one topology
- ✅ Redistribute **RIP → EIGRP** and **static → EIGRP** on the ASBR
- ✅ Recognize **external EIGRP routes** (`D EX`, AD **170**)
- ✅ Understand the **5-value seed metric** (bandwidth, delay, reliability, load, MTU)
- ✅ Compare redistribution **with and without** a metric
- ✅ **Calculate** the redistributed metric by hand
- ✅ Use the metric to give one source **priority** over another
- ✅ Verify results from a router **inside the EIGRP domain** (Router1)

---

## 🗺️ Network Topology

```
                  EIGRP AS 1 (yellow)                                      RIPv2 (purple)

  L1 10.1.1.1/32                 L2 20.1.1.1/32                 L3 30.1.1.1/32
        │                              │                              │
   ┌─────────┐ gig0/0   gig0/0 ┌─────────┐ gig0/1   gig0/1 ┌─────────┐       🌸 STATIC (Null0)
   │ Router0 ├─────────────────┤ Router1 ├─────────────────┤ Router2 │       100.100.100.0/24
   │  1941   │ 1.1.1.1  1.1.1.2│  1941   │ 2.1.1.1  2.1.1.2│  1941   │       100.100.150.0/24
   └────┬────┘     1.1.1.0/24  └─────────┘     2.1.1.0/24  └────┬────┘       100.100.200.0/24
        │ gig0/1                                                │ gig0/0
        │ 192.168.1.1                                           │ 192.168.2.1
   ┌────┴────┐                                             ┌────┴────┐
   │ Switch0 │                                             │ Switch2 │
   └─┬──┬──┬─┘                                             └─┬──┬──┬─┘
   PC1 PC0 PC2   192.168.1.0/24                          PC6 PC7 PC8   192.168.2.0/24

                                            ▲
                                  Router2 = ASBR  (EIGRP ⇄ RIP + static)
```

> 📷 Add your screenshot: `![Topology](images/topology.png)`

> 📝 **Naming note:** the Packet Tracer icons are *Router0 / Router1 / Router2*, while the CLI prompts in the screenshots show **R2** (Router1) and **R3** (Router2).

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Notes |
|:------:|:---------:|:----------:|:------|
| **Router0** | Loopback (L1) | 10.1.1.1/32 | Advertised in EIGRP |
| **Router0** | Gi0/0 | 1.1.1.1/24 | Link to Router1 |
| **Router0** | Gi0/1 | 192.168.1.1/24 | LAN 1 (Switch0) |
| **Router1** | Gi0/0 | 1.1.1.2/24 | Link to Router0 |
| **Router1** | Gi0/1 | 2.1.1.1/24 | Link to Router2 |
| **Router1** | Loopback1 (L2) | 20.1.1.1/32 | Connected |
| **Router2** | Gi0/1 | 2.1.1.2/24 | Link to Router1 |
| **Router2** | Loopback (L3) | 30.1.1.1/32 | Advertised in EIGRP |
| **Router2** | Gi0/0 | 192.168.2.1/24 | LAN 2 (Switch2) — **RIPv2** side |
| PC1 / PC0 / PC2 | NIC | 192.168.1.x/24 | LAN 1 |
| PC6 / PC7 / PC8 | NIC | 192.168.2.x/24 | LAN 2 |

---

## 🧱 Routing Domains

| Domain | Routers / Networks | Protocol |
|:------:|:-------------------|:--------:|
| 🟡 Core | Router0 · Router1 · Router2 · 1.1.1.0/24 · 2.1.1.0/24 · loopbacks · 192.168.1.0/24 | **EIGRP AS 1** |
| 🟣 Edge | 192.168.2.0/24 on Router2 | **RIPv2** |
| 🌸 Static | 100.100.100.0/24 · 100.100.150.0/24 · 100.100.200.0/24 | **Static → Null0** |

---

## ⚙️ Configuration

### 🔹 Static routes on Router2 *(shown in the topology)*

```bash
ip route 100.100.100.0 255.255.255.0 null0
ip route 100.100.150.0 255.255.255.0 null0
ip route 100.100.200.0 255.255.255.0 null0
```

### 🔹 Redistribution on Router2 (the ASBR)

**Stage ①** — RIP into EIGRP with a metric (static redistributed without one)

```bash
router eigrp 1
 redistribute rip metric 1 1 1 1 1
 redistribute static
```

**Stage ②** — give the static routes the same metric

```bash
router eigrp 1
 redistribute static metric 1 1 1 1 1
```

**Stage ③** — change the RIP seed metric to make RIP preferred

```bash
router eigrp 1
 redistribute rip metric 5 5 5 5 5
do write memory
```

> ⚠️ The base configuration (EIGRP network statements, `router rip` / `version 2` on Router2, interface IPs) was already in place. Match it to your own running-config if it differs.

<details>
<summary><b>📜 Click to see the redistribute metric help exactly as shown in the lab</b></summary>

```
R3(config)# router eigrp 1
R3(config-router)# redistribute rip metric 1 ?
  <0-4294967295>  EIGRP delay metric, in 10 microsecond units
R3(config-router)# redistribute rip metric 1 1 ?
  <0-255>         EIGRP reliability metric where 255 is 100% reliable
R3(config-router)# redistribute rip metric 1 1 1 ?
  <1-255>         EIGRP Effective bandwidth metric (Loading) where 255 is 100% loaded
R3(config-router)# redistribute rip metric 1 1 1 1 ?
  <1-65535>       EIGRP MTU of the path
R3(config-router)# redistribute rip metric 1 1 1 1 1
```

</details>

---

## ✅ Verification — Stage by Stage

> All routing tables below are from **Router1** (inside the EIGRP domain, `show ip route`).

### 🟡 Stage 1: Static Without Metric vs RIP With Metric

```
Router1# show ip route
     1.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       1.1.1.0/24 is directly connected, GigabitEthernet0/0
     2.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       2.1.1.0/24 is directly connected, GigabitEthernet0/1
     10.0.0.0/32 is subnetted, 1 subnets
D       10.1.1.1/32 [90/130816] via 1.1.1.1, 00:25:12, GigabitEthernet0/0
     20.0.0.0/32 is subnetted, 1 subnets
C       20.1.1.1/32 is directly connected, Loopback1
     30.0.0.0/32 is subnetted, 1 subnets
D       30.1.1.1/32 [90/130816] via 2.1.1.2, 00:24:38, GigabitEthernet0/1
     100.0.0.0/24 is subnetted, 3 subnets
D EX    100.100.100.0/24 [170/25856] via 2.1.1.2, 00:23:45, GigabitEthernet0/1    ◄── 🌸 static, no metric
D EX    100.100.150.0/24 [170/25856] via 2.1.1.2, 00:23:45, GigabitEthernet0/1
D EX    100.100.200.0/24 [170/25856] via 2.1.1.2, 00:23:45, GigabitEthernet0/1
D       192.168.1.0/24 [90/28416] via 1.1.1.1, 00:25:12, GigabitEthernet0/0
D EX    192.168.2.0/24 [170/2560000512] via 2.1.1.2, 00:01:37, GigabitEthernet0/1 ◄── 🟣 RIP, metric 1 1 1 1 1
```

🧠 **What this shows**

- The static routes were redistributed **without a metric** → the simulator assigned an automatic value: **25856**
- The RIP route used the **seed metric `1 1 1 1 1`** → **2,560,000,512** (a huge number — bandwidth of 1 Kbps)
- **Internal** EIGRP routes (`D`, AD 90) vs **external** routes (`D EX`, AD 170) are clearly separated

---

### 🟢 Stage 2: Static Gets a Metric Too → Everything Equal

```
R3(config)# router eigrp 1
R3(config-router)# redistribute static metric 1 1 1 1 1
R3(config-router)# do wr
```

```
Router1# show ip route
D EX    100.100.100.0/24 [170/2560000512] via 2.1.1.2, 00:01:03, GigabitEthernet0/1   ◄── 🔥 same metric
D EX    100.100.150.0/24 [170/2560000512] via 2.1.1.2, 00:01:03, GigabitEthernet0/1
D EX    100.100.200.0/24 [170/2560000512] via 2.1.1.2, 00:01:03, GigabitEthernet0/1
D       192.168.1.0/24 [90/28416] via 1.1.1.1, 00:28:21, GigabitEthernet0/0
D EX    192.168.2.0/24 [170/2560000512] via 2.1.1.2, 00:04:46, GigabitEthernet0/1     ◄── 🔥 same metric
```

🧠 **What this shows:** with `metric 1 1 1 1 1` on **both** sources, every redistributed route now has **`2560000512`**. No source is preferred — a clean baseline.

---

### 🔴 Stage 3: RIP Metric `5 5 5 5 5` → RIP Gets Priority ⭐

```
R3(config)# router eigrp 1
R3(config-router)# redistribute rip metric 5 5 5 5 5
R3(config-router)# do wr
```

```
Router1# show ip route
D EX    100.100.100.0/24 [170/2560000512] via 2.1.1.2, 00:03:55, GigabitEthernet0/1   ◄── 🌸 static: 2,560,000,512
D EX    100.100.150.0/24 [170/2560000512] via 2.1.1.2, 00:03:55, GigabitEthernet0/1
D EX    100.100.200.0/24 [170/2560000512] via 2.1.1.2, 00:03:55, GigabitEthernet0/1
D EX    192.168.2.0/24 [170/512001536]    via 2.1.1.2, 00:01:19, GigabitEthernet0/1   ◄── 🟣 RIP: 512,001,536  ⭐ LOWER = BETTER
```

🧠 **What this shows**

| Source | Seed metric | Router1 sees | Priority |
|:------:|:-----------:|:------------:|:--------:|
| 🌸 Static | `1 1 1 1 1` | **2,560,000,512** | Lower priority |
| 🟣 RIP | `5 5 5 5 5` | **512,001,536** | ⭐ **Higher priority** |

A **bigger bandwidth value** in the seed metric means a **smaller** EIGRP metric (`10⁷ ÷ bandwidth`) — and **lower metric wins**.

---

### 🧾 Evidence Summary

| # | Screenshot | What it proves |
|:-:|:-----------|:---------------|
| 1 | Topology | EIGRP AS 1 + RIPv2 + 3 static routes, Router2 is the ASBR |
| 2 | Router2 redistribute help | The 5 seed-metric fields and their units |
| 3 | Router1 route table (stage 1) | Static without metric = 25856, RIP = 2560000512 |
| 4 | Router2 `redistribute static metric 1 1 1 1 1` | Giving static a metric |
| 5 | Router1 route table (stage 2) | All redistributed routes equal at 2560000512 |
| 6 | Router2 `redistribute rip metric 5 5 5 5 5` | Changing the RIP seed to gain priority |
| 7 | Router1 route table (stage 3) | RIP = 512001536 vs static = 2560000512 |

### 📷 Add These Screenshots to Complete the Lab

```bash
show ip protocols                      # on Router2: shows "redistributing rip, static"
show ip route                          # on Router0: 100.100.x.0 and 192.168.2.0 learned as D EX
show ip eigrp topology 192.168.2.0/24  # shows external data: originated by Router2, protocol RIP
ping 192.168.2.2                       # from PC1 (192.168.1.x) → LAN 2
```

---

## 🧮 Metric Calculations

Redistributed (external) routes use the same EIGRP formula, with the **seed values you typed**:

```
Metric = 256 × ( 10⁷ ÷ bandwidth(Kbps)  +  delay )
                  └── from seed metric        └── seed delay + link delay
```

| Seed metric | Calculation | Result | Router shows |
|:-----------:|:------------|:------:|:------------:|
| `1 1 1 1 1` | 256 × ( 10,000,000 ÷ 1 + (1 + 1) ) = 256 × 10,000,002 | **2,560,000,512** | ✅ `2560000512` |
| `5 5 5 5 5` | 256 × ( 10,000,000 ÷ 5 + (5 + 1) ) = 256 × 2,000,006 | **512,001,536** | ✅ `512001536` |

```
 Seed bandwidth 1 Kbps  → 10⁷ ÷ 1 = 10,000,000   → huge metric  → LOW priority
 Seed bandwidth 5 Kbps  → 10⁷ ÷ 5 =  2,000,000   → 5× smaller   → HIGH priority
```

> 💡 The extra `+1` in the delay is the link delay added on Router1's receiving interface.
> The seed fields are **bandwidth · delay · reliability · load · MTU**; with default K-values only **bandwidth** and **delay** affect the metric.

---

## 🧩 `redistribute` Command Breakdown

```
redistribute rip metric 5 5 5 5 5
     │         │     │   │ │ │ │
     │         │     │   │ │ │ └── MTU                     (1 – 65535)
     │         │     │   │ │ └──── Load / loading          (1 – 255)
     │         │     │   │ └────── Reliability             (0 – 255)
     │         │     │   └──────── Delay (10 µs units)     (0 – 4294967295)
     │         │     └──────────── Bandwidth (Kbps)
     │         └────────────────── keyword: seed metric follows
     └──────────────────────────── source protocol
```

| Field | Order | Affects the metric? |
|:------|:-----:|:-------------------:|
| Bandwidth (Kbps) | 1st | ✅ Yes |
| Delay (10 µs) | 2nd | ✅ Yes |
| Reliability | 3rd | ❌ No (K4 = 0) |
| Load | 4th | ❌ No (K2 = 0) |
| MTU | 5th | ❌ No (not used in the metric) |

---

## 🔍 Internal vs External EIGRP Routes

| | 🟡 Internal (`D`) | 🔴 External (`D EX`) |
|:--|:----------------:|:--------------------:|
| Origin | Inside EIGRP AS | Redistributed from another source |
| Administrative Distance | **90** | **170** |
| Examples here | 10.1.1.1/32 · 30.1.1.1/32 · 192.168.1.0/24 | 100.100.x.0/24 · 192.168.2.0/24 |
| Metric | Calculated from the real path | **Seed metric you configured** + path |

> 🧭 EIGRP trusts **internal** routes more (AD 90) than **external** ones (AD 170).

---

## 🛠️ Useful Commands

```bash
show ip route                          # D = internal, D EX = external
show ip route eigrp                    # EIGRP routes only
show ip protocols                      # redistribution sources and metrics
show ip eigrp topology                 # FD / RD for every prefix
show ip eigrp topology 192.168.2.0/24  # external data: protocol, originator, metric
show running-config | section eigrp    # exact redistribute lines
```

---

## 🧰 Troubleshooting Guide

| Symptom | Likely Cause | Fix |
|:--------|:-------------|:----|
| Redistributed routes missing in EIGRP | No seed metric (and no `default-metric`) for the source | Add `metric b d r l m` or `default-metric b d r l m` |
| Routes appear but all look "worse" than expected | Seed bandwidth too small (`1` Kbps) | Raise the bandwidth value to lower the metric |
| RIP route never appears | RIP not running or not advertising 192.168.2.0 | Check `router rip` / `version 2` / network statement |
| Static route not redistributed | `ip route … null0` missing or `redistribute static` missing | Verify both |
| Routes flap or loop | Two-way redistribution without filtering | Use route-maps + tags (see Advanced) |
| Wrong route type in table | Expecting `D` but seeing `D EX` | Normal — redistributed routes are **external** |

---

## 🚀 Advanced Topics

### 🔹 `default-metric` — set the seed once for all sources

```bash
router eigrp 1
 default-metric 1000 100 255 1 1500
 redistribute rip
 redistribute static
```

### 🔹 Filter what gets redistributed (route-map)

```bash
ip prefix-list STATIC-NETS seq 5 permit 100.100.100.0/24
route-map STATIC-TO-EIGRP permit 10
 match ip address prefix-list STATIC-NETS
 set metric 1000 100 255 1 1500
router eigrp 1
 redistribute static route-map STATIC-TO-EIGRP
```

### 🔹 Two-way redistribution (only when RIP needs EIGRP routes)

```bash
router rip
 redistribute eigrp 1 metric 2        # RIP metric is hop count
```
> ⚠️ With **two ASBRs** or **two-way** redistribution, always use **route tags + route-maps** to prevent routing loops.

### 🔹 Why static routes to `Null0`?

Pointing a prefix at `Null0` creates a **blackhole/summary anchor**: the router can advertise the prefix (e.g., to a whole domain) while discarding traffic for networks that don't exist behind it.

---

## 🎓 Key Takeaways

1. 🔸 **Redistribution** shares routes between different routing sources — done on the **ASBR**
2. 🔸 Redistributed routes appear in EIGRP as **`D EX`** with **AD 170**
3. 🔸 Internal EIGRP routes (`D`) keep **AD 90**
4. 🔸 The **seed metric** has 5 fields: **bandwidth, delay, reliability, load, MTU**
5. 🔸 Only **bandwidth and delay** change the metric with default K-values
6. 🔸 `metric 1 1 1 1 1` → **2,560,000,512** · `metric 5 5 5 5 5` → **512,001,536**
7. 🔸 **Bigger seed bandwidth → smaller metric → higher priority**
8. 🔸 Giving all sources the same metric makes them equal; changing one sets **priority**
9. 🔸 Always verify from a router **inside** the EIGRP domain (Router1 here)
10. 🔸 Use `default-metric`, **route-maps** and **tags** in production

---

## 🧪 Challenges

1. Remove `metric` from `redistribute rip` — does the RIP route still appear? Why or why not?
2. Set `metric 1000 100 255 1 1500` and predict the new metric **before** checking
3. Redistribute only **one** static prefix using a route-map
4. Add a second RIP router and enable two-way redistribution **with route tags**
5. Check `show ip eigrp topology 192.168.2.0/24` — what external data does it show?
6. Ping from PC1 (192.168.1.x) to PC6 (192.168.2.x) and trace the path

---

## 📁 Repository Structure

```
📦 EIGRP-Redistribution
 ┣ 📜 README.md
 ┣ 📂 images
 ┃ ┣ 🖼️ topology.png
 ┃ ┣ 🖼️ 01-redistribute-rip-metric-help.png
 ┃ ┣ 🖼️ 02-router1-static-no-metric.png
 ┃ ┣ 🖼️ 03-redistribute-static-metric.png
 ┃ ┣ 🖼️ 04-router1-all-metrics-equal.png
 ┃ ┣ 🖼️ 05-redistribute-rip-metric-5.png
 ┃ ┗ 🖼️ 06-router1-rip-priority.png
 ┗ 📂 lab-file
   ┗ 📦 EIGRP-Redistribution.pkt
```

---

## 🔗 EIGRP Lab Series

| # | Lab |
|:-:|:----|
| 1 | EIGRP Successor – Single Path |
| 2 | EIGRP Successor – 2 Paths (Equal-Cost Load Balancing) |
| 3 | EIGRP Unequal-Cost Load Balancing & Variance |
| 4 | EIGRP MD5 Authentication |
| 5 | EIGRP Metric Calculation |
| 6 | **EIGRP Redistribution (RIP + Static)** ◄── *you are here* |

`EIGRP` · `Redistribution` · `RIPv2` · `Static Routes` · `ASBR` · `Seed Metric` · `External Routes` · `Route Priority`

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

---

<div align="center">

⭐ **If this lab helped you, please star the repository!** ⭐

*Part of my ongoing Cisco Packet Tracer lab series — routing, switching, security & more.*

</div>
