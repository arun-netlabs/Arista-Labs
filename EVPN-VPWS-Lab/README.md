# EVPN VPWS Lab — Segment Routing MPLS with EVPN VPWS on Arista cEOS

## Overview

This lab builds a **Segment Routing MPLS** backbone with **EVPN VPWS** (Virtual Private Wire Service) to deliver point-to-point Layer 2 connectivity between two customer sites — all running on Arista cEOS in containerlab.

EVPN VPWS (RFC 8214) provides a point-to-point Ethernet pseudowire signaled via BGP EVPN and transported over an SR-MPLS core. No LDP, no RSVP-TE — just ISIS with segment routing extensions.

For a detailed walkthrough of the configuration, packet walks, and Wireshark analysis, see the full article on LinkedIn:
**[LinkedIn Article — SR-MPLS with EVPN VPWS](#)** *(replace with your LinkedIn article URL)*

## Topology

```
                                UPPER PATH
                        P1 ────── BGP-RR ────── P4
                       /│ \                    / │ \
                      / │  \                  /  │  \
CE-1-A ── PE-1 ─────/  │   \────────────────/   │   \───── PE-2 ── CE-1-B
             │  \       │                        │       /
             │   P3 ──  P2 ──────────────────────+  ── P5
             │      \  / │                          /
             │       P6  │                         /
             │        \  │────────────────────────/
             └─────────\─┘

VPWS Pseudowire: CE-1-A ←── Ethernet over SR-MPLS ──→ CE-1-B
```

## Nodes

| Node | Role | Router-ID | Node SID | MPLS Label |
|------|------|-----------|----------|------------|
| PE-1 | Provider Edge | 1.1.1.1 | 1 | 900001 |
| PE-2 | Provider Edge | 2.2.2.2 | 2 | 900002 |
| P1 | Core | 11.11.11.11 | 11 | 900011 |
| P2 | Core | 22.22.22.22 | 22 | 900022 |
| P3 | Core | 33.33.33.33 | 33 | 900033 |
| P4 | Core | 44.44.44.44 | 44 | 900044 |
| P5 | Core | 55.55.55.55 | 55 | 900055 |
| P6 | Core | 66.66.66.66 | 66 | 900066 |
| BGP-RR | Route Reflector | 77.77.77.77 | 77 | 900077 |
| CE-1-A | Customer (Site A) | 172.16.1.1 | — | — |
| CE-1-B | Customer (Site B) | 172.16.1.2 | — | — |

## Protocols & Services

- **Underlay**: ISIS CORE (Level-2, IPv4) with Segment Routing MPLS
- **Overlay**: iBGP EVPN via Route Reflector (MPLS encapsulation)
- **Service**: EVPN VPWS pseudowire between PE-1 and PE-2 (RFC 8214)
- **Transport**: SR-MPLS labels (SRGB 900000–965535)

## Prerequisites

- [containerlab](https://containerlab.dev) installed
- Arista cEOS Docker image (`ceos:latest`)

## Deploy the Lab

```bash
# Clone the repo
git clone https://github.com/arun-netlabs/Arista-Labs.git
cd Arista-Labs/EVPN-VPWS-Lab

# Deploy
sudo containerlab deploy -t topology.yaml

# Verify all nodes are running
sudo containerlab inspect -t topology.yaml
```

## Access the Nodes

```bash
# CLI access to any node
docker exec -it clab-sr-mpls-lab-PE-1 Cli
docker exec -it clab-sr-mpls-lab-BGP-RR Cli
docker exec -it clab-sr-mpls-lab-CE-1-A Cli
```

## Verify the Lab

```bash
# From CE-1-A — ping CE-1-B through the VPWS pseudowire
docker exec clab-sr-mpls-lab-CE-1-A Cli -p 15 -c "ping 172.16.1.2"

# Check ISIS neighbors on PE-1
docker exec clab-sr-mpls-lab-PE-1 Cli -p 15 -c "show isis neighbors"

# Check SR prefix segments
docker exec clab-sr-mpls-lab-PE-1 Cli -p 15 -c "show isis segment-routing prefix-segments"

# Check BGP EVPN session
docker exec clab-sr-mpls-lab-PE-1 Cli -p 15 -c "show bgp evpn summary"

# Check VPWS pseudowire status
docker exec clab-sr-mpls-lab-PE-1 Cli -p 15 -c "show patch panel detail"
```

## Destroy the Lab

```bash
sudo containerlab destroy -t topology.yaml
```

## Repository Structure

```
Arista-Labs/
└── EVPN-VPWS-Lab/
    ├── README.md
    ├── topology.yaml
    └── configs/
        ├── PE-1.cfg
        ├── PE-2.cfg
        ├── P1.cfg ~ P6.cfg
        ├── BGP-RR.cfg
        ├── CE-1-A.cfg
        └── CE-1-B.cfg
```
