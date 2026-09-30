# Hardware

These are the **reference options, not final designs.** All prices are rough 2026 estimates in US$ for small quantities, and they vary a lot by country. Please contribute local prices and suppliers.

## 1. Node

A node is a **heartbeat-only beacon**: it wakes up, transmits about 12 bytes and goes back to sleep. It has no sensors and no receiver. Because deployments need hundreds to thousands of nodes, **unit cost and battery life matter most**.

### Requirements
- Runs for **3 years or more** on a primary battery at the deployment's heartbeat interval (30–60 s)
- Sleep current in the **single-digit µA**
- Transmit only, never receive
- Waterproof (IP67 or better)
- **Detachment means silence**: breakaway mount plus breakaway power (see below)
- Target cost **under US$10 in volume** (1,000+), including enclosure and battery

### MCU + radio options

| Option | Notes |
|---|---|
| **STM32WL-based module** (e.g. RAK3172, Seeed Wio-E5) or a bare STM32WL chip on a custom PCB | MCU and LoRa on one chip, µA-level sleep. **Recommended.** A bare chip on a custom PCB is cheapest at volume. |
| ATtiny / low-power MCU + SX1262 | Cheap parts, slightly more complex |
| ESP32 + LoRa dev boards | For bench prototyping only. **Far too power-hungry** for field nodes. |

### Breakaway power (recommended)
A detached node that floats could keep sending heartbeats. To make sure **detached means silent**, route the battery through the mount:
- a **pull-pin or breakaway contact** tied to the mount (like a lanyard kill switch), **or**
- a reed switch that cuts power when the node leaves a magnet on the mount.

This costs a few cents and makes the signal unambiguous. Submersion also silences a node, because water blocks the radio.

### Power budget (example: STM32WL, SF7/125 kHz, 12-byte packet, T = 30 s)

| Item | Estimate |
|---|---|
| Sleep | ~3 µA × 24 h ≈ 0.07 mAh/day |
| Heartbeat TX | ~40 mA × ~0.045 s (plus wake overhead ≈ 2 mAs per beat) × 2,880/day ≈ 1.6 mAh/day |
| **Total** | **≈ 1.7 mAh/day ≈ 620 mAh/year** |

With T = 60 s this is about half. 2 × AA lithium cells (~3,000 mAh) last **about 4 years at T = 30 s** in theory. Temperature and battery ageing reduce that, so **measure real current and confirm**. Cheaper cells (e.g. alkaline) are viable at longer intervals.

### Enclosure and mount
- A small IP67 box, or potted electronics in PVC pipe. At volume, consider an injection-moulded or 3D-printed housing.
- Antenna pointing upward. Conformal coating on the PCB.
- Mount: a stake, clamp or strap with a **deliberately weak link** (sized zip-tie, shear pin or magnet), combined with the breakaway power contact above.
- Record the **elevation of the node itself** (ground + mounting height) accurately in the registry.
- A label with the project name, a "please do not remove" message in the local language and a contact number.

### Provisioning at scale
A simple provisioning tool or jig flashes each node with its ID, key and interval, prints a QR label and adds the node to the registry when it is surveyed in the field.

## 2. Station

### Requirements
- Always listening for LoRa, **on many channels at once** for large node counts
- Accurate time (GNSS or NTP)
- Tracks when each node's heartbeat is due, and batches arrivals and misses to base
- Backhaul to base, with store-and-forward
- **Local siren output** for autonomous alarms
- **5 days or more of autonomy** without sun, to cover monsoon cloud cover

### Two station classes

| Class | Radio | Host | Nodes (at T = 30–60 s, see [ARCHITECTURE.md §5](ARCHITECTURE.md#5-radio-capacity-important-at-this-scale)) | Average power | Rough cost |
|---|---|---|---|---|---|
| **Small** | Single-channel SX1262 | ESP32-S3 | ~30–70 | ~0.3–0.6 W | US$80–150 |
| **Large** | **8-channel SX1302/SX1303 concentrator** (e.g. RAK2287, WM1302) | Raspberry Pi Zero 2 W / CM4 / similar Linux board | Several hundred | ~2–3 W | US$150–250 |

The large class needs a Linux host to run the concentrator, which justifies a Raspberry Pi *at the station* in this case. It needs a correspondingly bigger solar and battery system (see below).

### Backhaul
- LTE Cat-1 / Cat-M / NB-IoT module (e.g. SIM7600, A7670, SIM7080G). **Avoid 2G/3G-only modules.**
- LoRa relay to a neighbouring station
- Optional satellite short-burst-data modem for remote sites

### Power sizing (worked examples)

```
                 Small station (0.5 W)          Large station (2.5 W)
Load/day:        12 Wh                          60 Wh
5-day autonomy:  60 Wh usable                   300 Wh usable
Battery (80%):   ≈ 75 Wh → 12.8 V 6 Ah          ≈ 375 Wh → 12.8 V 30 Ah
Panel:           12/(3 sun-h × 0.7) ≈ 6 W       60/(3 × 0.7) ≈ 29 W
                 → use 20 W                     → use 60–80 W
```

- Use an **MPPT charge controller** suited to LiFePO4.
- The **BMS must stop charging below 0 °C**. This is critical at mountain and GLOF sites.
- Protect against surges and lightning: ground the mast and put surge protection on the antenna feed.

### Siren / beacon output
A relay or MOSFET output drives a 12 V siren and strobe, powered from the station battery. A separate siren station can be placed in the village and triggered over LoRa.

## 3. Base station

This tier doesn't need special hardware:
- A small cloud VM, **or** a local PC or Raspberry Pi 4/5 with a UPS
- Ideally one local and one cloud instance for redundancy
- An SMS gateway (GSM modem or SMS API) for alerts

## 4. Rough BOM summary

| Unit | Estimate |
|---|---|
| Node (volume) | under US$10 |
| Small station (without siren) | US$80–150 |
| Large station incl. solar/battery | US$300–500 |
| Siren + strobe | US$20–60 |
| Example pilot: 300 nodes + 2 large stations + 1 small | ≈ US$3,500–4,500 |

## 5. Radio regulations

LoRa uses unlicensed ISM bands that **differ by country** (e.g. EU868, US915, AS923, IN865). Each band has limits on transmit power, duty cycle and dwell time. **Check your national telecom regulator before you deploy.** Firmware must allow the region to be configured and must follow its limits.
