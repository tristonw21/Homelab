# Home Rack Setup Documentation

## Purpose

This document serves as a **high-level, stable overview** of the **hardware inventory, physical layout, power design, and network interconnections** for my home rack.

It is intentionally **service-agnostic**:

* Software, services, and VM/container workloads are expected to change frequently
* This document focuses on what is *physically connected*, *powered*, and *networked*

Detailed service documentation, Proxmox VM layouts, and application configs are tracked **separately**.

---

## Location & Environment

* **Location:** Kingfisher, Oklahoma
* **Environment:** Home / homelab
* **Primary Goals:**

  * Learn networking and systems administration
  * Host internal services and limited external services
  * Balance capability with **power efficiency**, **noise**, and **cost**

---

## Rack Overview

* **Rack:** 24U Dell APC NetShelter (enclosed)
* **Form Factor:** Standard 19-inch
* **Cooling:** Passive rack airflow (room-dependent)
* **Noise Considerations:** Important (home environment)

### Physical Signal & Power Flow (High-Level)

* **ISP:** Pioneer Internet
* **ONT / Gateway:** Gigaspire Blast U6X
* **Core Rack:** Dell 24U NetShelter
* **Primary Power:** Rack-mounted Dell-APC Smart-UPS 1500VA
* **Power Distribution:** Rear-mounted PDU fed by UPS

---

## Power Infrastructure

### UPS

* **Model:** Dell-APC Smart-UPS 1500VA (Rack-mounted)
* **Capacity:** 1500VA / 1000W
* **Placement:** Bottom of rack (weight-optimized)
* **Function:**

  * Battery-backed power for all rack equipment
  * Graceful shutdown capability

### Power Distribution

* **PDU:** Rear-mounted rack PDU
* **Feed Source:** UPS output
* **Role:** Single-point power distribution to all devices

### Power Design Philosophy

* Centralized UPS → single PDU → all rack loads
* Designed for safe shutdown, not long-duration runtime
* High-draw systems intended for intermittent use

---

## Compute Hardware (Hardware-Focused Overview)

### Dell PowerEdge R820

* **Source:** Local pickup / resale listing
* **Role (Current):** Offline / evaluation
* **Potential Roles:**

  * Proxmox hypervisor
  * High-capacity NAS (episodic)
  * Compute-heavy batch workloads

#### Hardware Specification

* 4× Intel Xeon E5-2680 CPUs (8 cores each)
* CPU riser included
* 256 GB DDR3 ECC memory (1333 MHz)
* 16× 2.5" hot-swap drive bays
* PERC H710 RAID controller
* 4× 1 GbE NICs
* 2× 1100W redundant power supplies
* iDRAC Enterprise licensed

#### Storage Configuration

* **Installed Drives:** 2× drives (user-supplied)
* **Array:** RAID 1
* **Usable Capacity:** ~230 GB
* **Purpose:** Boot + core services only (not bulk storage)

#### Software State (Non-Authoritative)

* **Current Hypervisor:** Proxmox VE
* **Note:** Software stack is expected to change and is not considered canonical in this document

#### Notes

* Server originally shipped with **no drives**
* Storage intentionally minimal due to power and role constraints

#### Known Constraints

* Extremely high idle and load power draw
* Significant fan noise under load
* Best suited for non-24/7 workloads

---

## Networking (Physical & Logical Connectivity)

### Core Switching

* **Switch:** Cisco Catalyst 3750G
* **Ports:** 48× Gigabit Ethernet
* **PoE:** Supported
* **Role:** Primary rack aggregation switch

### Edge & Internet

* **ISP:** Pioneer Internet
* **Gateway:** Gigaspire Blast U6X
* **Uplink Medium:** Cat6 Ethernet
* **Internet Speed:** Gigabit fiber (Gigabit Ethernet handoff)

### Networking Goals

* VLAN segmentation (management / services / lab)
* Transition path to >1G internal networking
* Eventual rack-mounted Layer 3 capable core

---

## Management & Control

### Out-of-Band Management

* **iDRAC:** Enabled on PowerEdge R820

### Remote Access & Security

* **Gatekeeper-Alpha:** Raspberry Pi 3 (Edge Gateway)
* **Access Model:** Zero-trust, mesh-based (no WAN ports)
* **Remote Management:** SSH over Tailscale

---

## Storage (Current & Planned)

* **Current State:** No dedicated NAS in production
* **Future Ideas:**

  * NAS hosted on R820 (intermittent use)
  * Lower-power dedicated NAS hardware
  * ZFS-based storage under Proxmox

---

## Smart Home & Automation Integration

* **Home Assistant:** Planned / partially deployed
* **Integration Targets:**

  * UPS status monitoring
  * Network uptime awareness
  * Automated server wake/sleep

---

## Expansion Ideas

### Short Term

* Measure actual UPS and server power draw
* Finalize Proxmox deployment locations
* Improve rack cable management

### Mid Term

* VLAN rollout on Cisco 3750G
* Dedicated low-power always-on server
* NAS strategy decision (R820 vs new hardware)

### Long Term

* Replace R820 with modern, efficient compute
* Dedicated NAS appliance
* 2.5G / 10G-ready network core

---

## Edge Security Architecture: Gatekeeper-Alpha

### Hardware

* Raspberry Pi 3 Model B

### Operating System

* DietPi (Debian 12)

### Core Components

* **Tailscale (WireGuard):** Identity-based mesh networking
* **UFW:** Default-deny firewall
* **Fail2Ban:** Brute-force mitigation
* **Dropbear SSH:** Reduced attack surface

### Network Role

* Acts as subnet router for 192.168.1.0/24
* No exposed WAN ports
* Mesh IP-based access only

### Security Posture Summary

* Zero exposed services to public internet
* Kernel-level firewall enforcement
* Automated intrusion mitigation
* Minimal persistent logging

---

## Open Questions / Notes

* What services justify 24/7 operation?
* How much power draw is acceptable monthly?
* Which hardware should be sold vs retained?

---

## Visual Architecture (Hardware & Network Topology Only)

### Physical Power & Network Flow

```
[Pioneer ISP]
     │
     ▼
[Gigaspire Blast U6X]
     │  (Gigabit Ethernet)
     ▼
┌───────────────────────────┐
│   Dell 24U NetShelter     │
│                           │
│  ┌─────────────────────┐ │
│  │ Cisco 3750G Switch  │◄┼── Internal LAN / PoE
│  └─────────────────────┘ │
│            ▲              │
│            │              │
│  ┌─────────────────────┐ │
│  │ PowerEdge R820      │ │
│  │ Proxmox Host        │ │
│  └─────────────────────┘ │
│            ▲              │
│  ┌─────────────────────┐ │
│  │ Rear PDU            │ │
│  └─────────────────────┘ │
│            ▲              │
│  ┌─────────────────────┐ │
│  │ APC Smart‑UPS 1500  │ │
│  └─────────────────────┘ │
└───────────────────────────┘
```

### Logical Security & Access Topology

```
          Remote Devices
                │
         (Tailscale Mesh)
                │
        ┌────────────────┐
        │ Gatekeeper‑Alpha│  Raspberry Pi 3
        │  Zero‑Trust Edge│
        └────────────────┘
                │  (Subnet Route)
        ────────┼────────────────────
                │
        Internal Trusted LAN (192.168.1.0/24)
                │
   ┌────────────┴────────────┐
   │ Cisco 3750G Switch       │
   └────────────┬────────────┘
                │
        Proxmox / Services / Lab
```

### Rack Unit (U) Layout – Detailed (Bottom → Top)

```
U24   Rack-mounted KVM
U23   ───────────────────────────
U22   Shelf – Gaming laptop (Proxmox)
U21   ───────────────────────────
U20   Shelf
U19   Half-depth shelf (shared)
U18   Half-depth shelf – Raspberry Pi nodes
U17   Half-depth shelf (air gap for switch)
U16   Cisco Catalyst 3750G (48p PoE)
U15   ───────────────────────────
U14   ───────────────────────────
U13   ───────────────────────────
U12   ───────────────────────────
U11   ───────────────────────────
U10   ───────────────────────────
U09   PowerEdge R820 (2U – upper)
U08   PowerEdge R820 (2U – lower)
U07   ───────────────────────────
U06   ───────────────────────────
U05   ───────────────────────────
U04   Shelf
U03   ───────────────────────────
U02   APC Smart-UPS 1500VA (2U – upper)
U01   APC Smart-UPS 1500VA (2U – lower)

Rear of Rack:
- Vertical PDU mounted to rear frame (not consuming U-space)
```

U24 ───────────────────────────
U23
U22   Patch / Cable Mgmt
U21
U20   Cisco 3750G (48p PoE)
U19
U18
U17   PowerEdge R820 (2U)
U16
U15
U14
U13
U12
U11
U10
U09
U08
U07
U06
U05
U04
U03
U02   Rear PDU (vertical)
U01   APC Smart‑UPS 1500VA

```

---

## Change Log
- Updated to confirm Gigabit Ethernet throughout
- Added Proxmox + RAID‑1 storage details
- Added physical, logical, and rack layout visuals

```
