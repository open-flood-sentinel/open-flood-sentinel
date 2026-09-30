# Architecture

This document describes how the system is put together and why. It is a **living design**. Challenge it in Discussions or open an issue.

## 1. Core idea

Nodes are **heartbeat-only beacons**. They measure nothing and send no event messages. Every node repeats one short packet at a regular interval, which means *"I am still here."*

A flood reveals itself through **the way nodes stop sending heartbeats**. Water submerges them (radio at these frequencies does not get out through water) or tears them away, so a flood produces a **contiguous region of silence** that:

- starts at the **lowest ground** near the channel,
- climbs to **higher elevations** as the water rises, and
- moves **downstream** at the speed of the flood.

Random failures (flat batteries, animals, theft, lost packets) are **isolated and scattered** in space and time. With hundreds or thousands of nodes, the two patterns are easy to tell apart, and the software treats isolated losses as noise.

## 2. Three tiers

| Tier | Where | Power | Job | Expected lifetime |
|---|---|---|---|---|
| **Node** | Across the valley or floodplain at known positions and elevations, hundreds to thousands of them | Primary battery (years) | Send a heartbeat every *T* seconds, nothing else | Until a flood destroys or detaches it (by design) |
| **Station** | High ground, out of the flood's reach, in radio line-of-sight of its nodes | Solar + LiFePO4 | Receive heartbeats, track when each node's next one is due, report arrivals and misses to base, sound a local alarm if isolated | Years |
| **Base station** | Somewhere safe with power and internet | Mains + UPS | Hold the state of every node, detect flood patterns, estimate speed, level, extent and ETA, issue alerts | Years |

## 3. Data flow

```
Nodes ──heartbeats──> Station ──(cellular | LoRa relay | satellite)──> Base ──> Alerts
                         │                                              │
                         └──> local siren (autonomous fallback)          └──> Live map, open data, simulation
```

1. The node wakes on its timer, sends a heartbeat of about 12 bytes, and goes back to sleep.
2. The station receives it and records the time, signal strength (RSSI) and signal-to-noise ratio (SNR). It also learns each node's rhythm so it knows **when the next heartbeat is due**.
3. When an expected heartbeat does not arrive within a grace window, the station records a **miss**.
4. The station sends heartbeat arrivals and misses to the base in batches every few seconds. While a flood pattern is building, it sends them immediately.
5. The base updates the state of each node and runs pattern detection and estimation across the whole field (see [ALGORITHMS.md](ALGORITHMS.md)).

## 4. Deployment layout

### Node field

Nodes are spread over the area that floods, **densest near the channel and on low ground**, and extend up the valley sides to the highest level worth measuring.

```
  elevation
      ▲       ·     ·     ·     ·     ·     ·      ← high ground: goes silent only in extreme floods
      │     ·   ·   ·   ·   ·   ·   ·   ·   ·
      │   · · · · · · · · · · · · · · · · · · ·
      │  ·~·~·~·~·~·~·~·~ channel ~·~·~·~·~·~·~·  ← densest here: first to go silent
      └──────────────────────────────────────────►  distance downstream (toward settlement)
```

- **Along the channel**: spacing of the order of 50–500 m. It sets the resolution of the speed estimate.
- **Up the valley sides**: nodes at several elevations at each location. They set the resolution of the level estimate, for example every 0.5–1 m of height near the channel.
- **Far enough upstream**: the most upstream nodes decide the maximum warning time (see [ALGORITHMS.md §8](ALGORITHMS.md#8-worked-example)).

### Deployment registry

Every node's ID, latitude/longitude, **elevation**, **distance along the flow path to each settlement**, station assignment and install date go into a versioned **registry**. The estimates are only as good as this data. Elevation from a phone GPS is not accurate enough. Use a survey, RTK GPS or a good elevation model plus a measured height above the ground.

### Station placement

- Well above the highest flood level on record, **plus a margin**.
- In line-of-sight of its nodes. Expect 2–15 km depending on terrain.
- **Overlapping coverage**: each node should ideally be heard by **2 or more stations**. The base can then tell "the node went silent" apart from "the station went down".
- Secure and accessible. A school, mosque, clinic or community member makes a good host.

### Breakaway mount

Mount nodes so that a flood detaches them easily, for example with a weak zip-tie, a shear pin or a magnetic mount. A detached node might **float and keep sending heartbeats**. To make sure detachment means silence, the recommended design runs the battery circuit **through the mount** (a pull-pin or breakaway contact). When the node is torn away, it loses power at once. See [HARDWARE.md](HARDWARE.md#1-node).

## 5. Radio capacity (important at this scale)

Thousands of nodes sending heartbeats on a shared channel will collide. Capacity must be designed in from the start.

For uncoordinated LoRa transmissions (the ALOHA model), the load on one channel and spreading factor is:

```
G = N × airtime / T          (N nodes, airtime per packet, heartbeat interval T)
P(collision) ≈ 1 − e^(−2G)
```

Example: 12-byte packet at SF7/125 kHz ≈ 41 ms airtime, T = 60 s:

| Nodes per channel | G | Packets lost to collisions |
|---|---|---|
| 35 | 0.024 | ~5% |
| 75 | 0.05 | ~10% |
| 150 | 0.10 | ~18% |

How to scale:
- **Many channels at once**: stations with large node counts use an **8-channel concentrator** (SX1302 class), not a single-channel radio.
- **Pseudo-random channel hopping** for each heartbeat (from `node_id` + `seq`), so two nodes never collide every time.
- **Random jitter** on each interval, so nodes don't lock into the same phase.
- **Many stations**, each serving its own area.
- **Choose *T* deliberately**: a shorter interval detects floods faster but increases collisions and uses more battery (see [ALGORITHMS.md §2](ALGORITHMS.md#2-when-is-a-node-lost)).

Packet loss is expected. The software learns each node's normal loss rate and allows for it.

## 6. Communication links

| Link | Technology | Notes |
|---|---|---|
| Node → Station | **Raw LoRa**, transmit only | Nodes never receive, which saves power. Must follow the region's band rules and duty-cycle limits. |
| Station → Base (primary) | **LTE Cat-1 / Cat-M / NB-IoT** | Avoid 2G/3G modules, which are being switched off in many countries. Cell towers often fail during disasters. |
| Station → Base (secondary) | **LoRa relay chain** between stations | Does not depend on telecom infrastructure. |
| Station → Base (optional) | **Satellite short-burst data** | For remote sites. Send only summaries. |

## 7. Failure modes

| Failure | What it looks like | How it is handled |
|---|---|---|
| One node dies (battery, animal, theft) | Isolated silence | Treated as noise. Maintenance ticket. |
| Packet collisions / interference | Occasional random misses | Per-node loss rate learned, several consecutive misses needed |
| **Station goes down** | All of its nodes go silent at the same moment, whatever their elevation | Station heartbeat lost too. Overlapping stations still hear the nodes. Not a flood pattern. |
| **Jamming / strong interference** | Mass silence at all elevations, raised noise floor at the station | Stations report noise floor. Operator alert. |
| Batteries ageing across the field | Slowly rising background loss | Battery voltage in heartbeats, replacement plan |
| Floating detached node | Keeps sending heartbeats from a new position | Breakaway power cut. RSSI jump flagged. |
| Cellular network down | Base goes blind | LoRa relay backhaul, **autonomous local siren** |
| Base server down | No alerts | Stations alarm locally. Base redundancy. |
| Spoofed or replayed heartbeats | Could **hide** a flood by faking live nodes | Message authentication code and frame counter (see [SECURITY.md](../SECURITY.md)) |
| Cold weather | LiFePO4 **must not be charged below 0 °C** | BMS with low-temperature charge cutoff |

## 8. Tiers are independent

- Each **station** runs a simplified version of the detection logic on its own nodes. If it detects a flood pattern **and** has lost contact with the base, it **sounds its local alarm**.
- A **base** that loses a station raises an operator alert straight away.

## 9. Future: simulation

Every heartbeat, miss and node position is logged in an open format. This supports:

1. **Testing before real floods.** A simulator generates heartbeats from thousands of virtual nodes, lets a synthetic flood cover them, and feeds the result through the real base software.
2. **Post-event replay.**
3. **Flood modelling.** The loss pattern is effectively a **measured map of flood extent over time**. Combined with terrain data, it can be used to calibrate flood models.

See [ROADMAP.md](ROADMAP.md).
