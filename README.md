# Open Flood Sentinel

**Low-cost, open-source early warning for floods, built from cheap sensors that are designed to be destroyed and to report it when they are.**

> ⚠️ **Project status: early concept / design phase.** Nothing here is ready to protect lives yet. Please read [SAFETY.md](docs/SAFETY.md) before you deploy anything.

---

## The idea

A flood destroys whatever is in its path, and no sensor that sits in the path is both cheap and indestructible. So we don't try to build one.

Our sensors are **cheap enough to lose**, and we **treat losing them as data**.

- **Nodes** are extremely simple, low-power LoRa beacons. They carry no sensors and send no messages. A node does exactly one thing: send a short **heartbeat** at a regular interval, which means *"I am still here."* We deploy **hundreds to thousands** of them across a river valley or floodplain upstream of settlements, each at a known position and elevation. They are mounted so that a flood easily **detaches** them.
- **Stations** sit on high ground, out of the flood's reach. They run on solar power with LiFePO4 battery backup, listen for the heartbeats and notice which ones stop arriving.
- **The base station** watches the whole field of nodes. The flood shows up as the **pattern in which nodes go silent**, and the base uses it to estimate **how fast the water is moving, how high it has risen, how much area it covers and how long it will take to reach people**.

A single node going silent means nothing. It is treated as noise: a dead battery, an animal, a lost packet. A flood looks completely different. It is a **contiguous wave of silence** that starts at the lowest ground, climbs to higher elevations and moves downstream over time. Having many nodes is what makes that pattern easy to recognise.

```
   UPSTREAM                                                  SETTLEMENT
   ~~~~~~~~~~~~~~~~~~~~~~~~~~~ river / valley ~~~~~~~~~~~~~~~~~~~~~~~~~>  🏘️

   · · · · · · · · · · · · · · · · · · · · · · · · · · · ·    (hundreds–thousands
   · · · · · · · · · · · · · · · · · · · · · · · · · · · ·     of heartbeat nodes
   · · · · · · · · · · · · · · · · · · · · · · · · · · · ·     at known positions
                                                                and elevations)
        \  heartbeats       |  heartbeats       /  heartbeats
         \                  |                  /
       (STATION 1)      (STATION 2)      (STATION 3)          (high ground,
          solar + LiFePO4, can sound a local siren on its own) out of reach)
              \             |             /
               \___ cellular / LoRa relay / satellite ___
                                   |
                            ( BASE STATION )  -> estimates speed, level, ETA
                                   |           -> alerts: SMS, sirens,
                                   v              loudspeakers, authorities
```

## What the system estimates

| Quantity | How |
|---|---|
| **Flood detected** | Many neighbouring nodes go silent together, far more than random failures would explain |
| **Flood front speed** | How fast the edge of the silent area moves downstream |
| **Water level** | The elevation dividing silent nodes from nodes still sending heartbeats in the same area |
| **Rate of rise** | How fast that dividing elevation climbs |
| **Flood extent** | A live map of the area where nodes have gone silent |
| **Time to reach the settlement (ETA)** | Distance still to travel ÷ front speed, plus a safety margin |

The maths is in [docs/ALGORITHMS.md](docs/ALGORITHMS.md).

## Target scenarios

- **Flash floods and hill torrents** in mountain valleys, where warning time is minutes to hours and official forecasts are coarse or missing.
- **Glacial lake outburst floods (GLOFs)**, which follow a known path and come with very little warning.
- **Small rivers and streams** upstream of villages that no national gauging network covers.

## Design principles

1. **Sacrificial, not indestructible.** Every node is cheap enough that losing it is expected.
2. **Keep nodes as simple as possible.** No sensors, no messages, just a heartbeat. That makes them cheap, long-lived and easy to make in large numbers.
3. **Rely on the pattern, not on any one node.** Single-node failures are expected and filtered out in software. Detection depends on how silence spreads across many nodes over space and time.
4. **Every tier can raise the alarm alone.** A station that loses its link to the base can still sound a local siren.
5. **Cheap and locally sourceable.** Target cost is under US$10 per node in volume and US$80–250 per station.
6. **Open everything.** Hardware, firmware, software, data formats and field reports are all open.
7. **Life-safety first.** We add to official warning systems and never replace them, and we are honest about limitations.

## Repository layout

```
open-flood-sentinel/
├── docs/              Architecture, algorithms, hardware, protocol, safety, roadmap
├── firmware/
│   ├── node/          Ultra-low-power heartbeat firmware
│   └── station/       Station firmware (LoRa RX, relay, local siren, backhaul)
├── base-station/      Server: ingestion, estimation, alerting, dashboard
├── hardware/          Schematics, PCBs, enclosures, mounts, BOMs
└── simulation/        Virtual nodes and flood scenarios for testing, later flood modelling
```

## Documentation

| Document | What's in it |
|---|---|
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | The three tiers, data flow, failure modes, deployment layout |
| [ALGORITHMS.md](docs/ALGORITHMS.md) | Detecting floods from silence patterns, estimating speed, level and ETA, alert levels, a worked example |
| [HARDWARE.md](docs/HARDWARE.md) | Reference hardware options, power budgets, enclosures, rough BOMs |
| [PROTOCOL.md](docs/PROTOCOL.md) | The heartbeat packet and station→base message format (draft) |
| [SAFETY.md](docs/SAFETY.md) | Life-safety principles, false alarms, working with authorities |
| [ROADMAP.md](docs/ROADMAP.md) | Phases from bench prototype to community pilot |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to help, whatever your skills |

## Get involved

You don't need to be an electronics engineer to help. We need people who work on:

- **Embedded / firmware**: ultra-low-power heartbeat nodes, multi-channel LoRa gateways
- **Hardware**: PCBs, waterproof enclosures, breakaway mounts, solar/battery design
- **Backend / data science**: detecting patterns in space and time, estimation, alerting, dashboards
- **Hydrology / civil engineering**: flood wave behaviour, site selection, validation
- **Field and community work**: local knowledge, deployment, training, translation
- **Docs and design**: guides, diagrams, build instructions in local languages

Start with [CONTRIBUTING.md](CONTRIBUTING.md) and the issues labelled [`good first issue`](../../issues?q=is%3Aopen+label%3A%22good+first+issue%22). Come and say hi in [Discussions](../../discussions).

## Licensing

| Part | License |
|---|---|
| Software and firmware | [Apache-2.0](LICENSE) |
| Hardware designs (`hardware/`) | [CERN-OHL-S-2.0](hardware/LICENSE) |
| Documentation (`docs/` and other `.md` files) | [CC BY-SA 4.0](docs/LICENSE) |

## Disclaimer

This project is provided **as is, without warranty of any kind**. A flood early warning system is a life-safety system. Do not rely on it as your only source of warning. Work with your local disaster management authority. See [SAFETY.md](docs/SAFETY.md).
