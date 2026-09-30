# Detection and estimation algorithms

This document covers how the base station (and, in a simpler form, each station) turns **heartbeats and their absence** into flood detection, speed, level, extent, ETA and alert levels. Everything here is a **draft for discussion**. Data scientists and hydrologists, please review and improve it.

## 1. Inputs

**Deployment registry**, one record per node:

| Field | Meaning |
|---|---|
| `node_id` | Unique ID |
| `lat`, `lon` | Position |
| `elev` | Elevation of the node itself, in metres (ground elevation + mounting height) |
| `d[s]` | Distance along the flow path from the node to settlement `s` (m) |
| `stations` | Stations expected to hear this node |

**Observations** from stations:
- `HEARD(node, station, rx_time, rssi, snr, seq, batt)`
- `MISSED(node, station, expected_time)`
- Station health: its own heartbeat, battery, noise floor

## 2. When is a node lost?

Each node sends a heartbeat every `T` seconds, with a small random jitter of `±j`. For each node, the station tracks:

- `t_last`: when it last heard the node
- `p_i`: the node's normal packet loss rate, learned over days or weeks
- the time the next heartbeat is due, `t_last + T`, with a grace window `g ≥ j`

A node is **lost** once it has missed `m` consecutive expected heartbeats **at every station that normally hears it**. Choose `m` so that a live node rarely looks lost by chance:

```
P(false loss) = p_i^m          e.g. p_i = 0.1, m = 3 → 0.001
```

**Loss time**: the node was destroyed somewhere between its last heartbeat and the first one it missed:

```
t_loss ∈ ( t_last , t_last + T ]      point estimate: t_last + T/2,  uncertainty ±T/2
```

**Detection delay** for one node is about `m × T + g`. With `T = 30 s` and `m = 3`, that is roughly 1.5–2 minutes. Having many nodes does not shorten this delay for any single node. What it does is:
- make the *pattern* statistically certain within a few losses, so the alert does not have to wait for more evidence;
- average out the ±T/2 timing uncertainty, which sharpens the speed estimate.

> **Choosing T** is the main design trade-off: a shorter interval means faster detection, but also more battery use and more radio congestion (see [ARCHITECTURE.md §5](ARCHITECTURE.md#5-radio-capacity-important-at-this-scale)). Pick it per deployment according to how much warning time the site needs.

## 3. Telling a flood from noise

Every lost node is classified by **the pattern around it**, not on its own.

| Pattern | Signature | Classification |
|---|---|---|
| **Random failure** | Isolated in space and time, at a background rate `λ` (losses per node per day) | Noise → maintenance ticket |
| **Station outage** | All nodes of one station lost within seconds of each other at *all* elevations, and the station heartbeat is lost too | Infrastructure fault |
| **Jamming** | Mass loss at all elevations, with a raised noise floor reported by the station | Infrastructure / security alert |
| **Flood** | Losses **clustered together**, **lowest elevations first**, spreading **upward and downstream** over time, while higher nodes nearby keep sending heartbeats | **Flood** |

### Scoring clusters

For each neighbourhood `k`, meaning a small area along the flow path with its nearby elevations:

1. **Count**: `n_k` = number of losses in a sliding window `W` (e.g. 10 min).
2. **Surprise**: compare with the number expected from random failure, `μ_k = N_k × λ × W`, using a Poisson tail probability `P(X ≥ n_k | μ_k)`. With `λ` ≈ 0.001/day, even 3–4 losses within minutes in one neighbourhood is extremely unlikely by chance.
3. **Elevation consistency**: in a flood, lost nodes are **lower** than nearby surviving nodes. The score is the fraction of lost nodes whose lower neighbours are also lost. It is close to 1 for a flood and close to random for noise.
4. **Growth**: the cluster keeps spreading (upward and/or downstream) over successive windows.

A neighbourhood is **flood-positive** when surprise, elevation consistency and growth all pass their configured thresholds. Station outages and jamming are excluded first, using station health data.

## 4. Water level and rate of rise

Within a flood-positive neighbourhood at time `t`, the water level `h` lies between the highest lost node and the lowest surviving node:

```
max{ elev_i : node i lost by t }   ≲   h   ≲   min{ elev_j : node j alive at t }
```

Use robust quantiles (e.g. the 90th percentile of lost elevations) so that one misplaced or broken node doesn't skew the result.

**Rate of rise** is the slope of `h(t)` over time. A fast rise, for example more than 1 m in 10 minutes, is a strong warning sign in itself.

## 5. Flood extent

The area covered by lost nodes in flood-positive neighbourhoods, taken together, gives a live **flood extent map** that updates as nodes go silent. Surviving nodes mark the edge.

## 6. Front speed and ETA

Divide the flow path into distance bins. For each bin `b`, let `T_b` be the time the **front arrived**. Use a robust value such as the median loss time of the first few low-elevation nodes lost in that bin, rather than the single earliest loss.

Fit distance against arrival time with a **robust regression** (e.g. Theil–Sen) across the bins. The slope is the front speed `v`.

```
ETA_nominal      = d_front / v
ETA_conservative = d_front / (v × SF)        SF = safety factor, default 1.5
```

where `d_front` is the distance from the most downstream flood-positive bin to the settlement.

**Always act on `ETA_conservative`.** Flood waves speed up in steep, narrow reaches, merge with tributaries and change behaviour. If there are not yet enough bins to fit a speed, use a **pre-configured worst-case speed** for that reach.

## 7. Alert levels

| Level | Trigger (defaults, configurable per deployment) | Action |
|---|---|---|
| **0 NORMAL** | Losses consistent with background noise | Routine health monitoring |
| **1 WATCH** | One neighbourhood flood-positive, **or** a surprising cluster that has not yet passed elevation consistency or growth | Notify operators |
| **2 WARNING** | Flood-positive with growth over 2 or more windows, or in 2 or more neighbourhoods | Notify responders and send a pre-alert to community focal points |
| **3 EVACUATE** | WARNING and (`ETA_conservative` < `T_evac` or `h` > `h_critical`) | Public alert: sirens, SMS, loudspeakers, authorities |

**Human in the loop vs automatic:** EVACUATE normally needs an operator to confirm. If `ETA_conservative` is shorter than `T_auto` (for example 30 minutes), the system escalates **automatically**. Each deployment must agree these thresholds with local authorities and communities.

## 8. Worked example

Village V. A node field stretches from 14 km to 4 km upstream, with about 600 nodes. `T = 30 s`, `m = 3`. The background loss rate works out to about 1 random loss per hour across the whole field.

| Time | What the base sees |
|---|---|
| 02:10–02:12 | 9 channel-level nodes lost in bins at 13.5–12 km. Poisson surprise ≈ 0. All lower neighbours also lost. → **WATCH** |
| 02:12–02:16 | Losses spread to nodes 1 m and then 2 m higher at 12–13 km. The nodes 3 m up are still alive. Level ≈ 2 m, rise ≈ 0.5 m/min. → **WARNING** |
| 02:16–02:22 | The front of losses reaches the 8 km bin. The robust fit gives about 4 km in 12 min. |

- Speed `v ≈ 5.6 m/s` (≈ 20 km/h)
- ETA from 8 km: `≈ 24 min` (nominal), `≈ 16 min` (conservative)
- 16 min < `T_auto` (30 min) → **EVACUATE, raised automatically**

The random background loss (about 1 per hour spread across the field) never forms a cluster like this, and the software ignores it.

The warning time depends on **how far upstream the node field reaches**. This must be planned per site from the expected flood speed and the time people need to evacuate.

## 9. Planned extensions

- **Discharge estimate** from level and surveyed channel geometry (Manning's equation, `Q = (1/n)·A·R^(2/3)·S^(1/2)`)
- **Rain-aware thresholds**: when heavy rain is forecast or measured, relax the WATCH criteria so alerts come sooner
- **Bayesian and learned detectors** trained on simulated and recorded events
- **Adaptive heartbeat**: stations cannot command nodes, which only transmit, but deployments may mix fast-interval nodes near the channel with slow-interval nodes higher up
