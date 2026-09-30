# Protocol (draft v0)

This is a draft wire format. Open an issue to propose changes before implementing them. Every change needs a version bump.

## 1. Node → Station: heartbeat (LoRa, binary, little-endian)

Nodes send **one message type only**: the heartbeat. They transmit and never receive. The packet is **12 bytes**.

| Offset | Size | Field | Notes |
|---|---|---|---|
| 0 | 1 | `ver` | Protocol version (0) |
| 1 | 4 | `node_id` | uint32, unique per node |
| 5 | 2 | `seq` | uint16 frame counter, persisted, used for replay protection and to count missed beats |
| 7 | 1 | `batt` | Battery voltage, `2000 + batt × 10` mV (range 2.00–4.55 V) |
| 8 | 4 | `mic` | Truncated AES-128-CMAC over bytes 0–7 with a per-node key |

### Timing rules
- Interval `T` (set per deployment at provisioning; default 30–60 s) with random jitter `±j` (default ±2 s), so nodes don't lock into the same phase.
- The channel for each heartbeat is chosen pseudo-randomly from `node_id` and `seq` across the deployment's channel plan, so that no two nodes collide every time.
- The first heartbeat after power-on is sent at once, with `seq` continuing from flash.
- All transmissions must respect the regional duty-cycle and dwell-time limits.

### Why `seq` matters
Gaps in `seq` show exactly how many heartbeats were missed. From these, stations learn each node's normal loss rate `p_i` (see [ALGORITHMS.md §2](ALGORITHMS.md#2-when-is-a-node-lost)).

## 2. Station → Base (MQTT over TLS)

A field of thousands of nodes would produce too many messages if every heartbeat were sent one by one. Stations **batch** them.

### Batches: `ofs/{deployment_id}/{station_id}/beats`
Sent every 10 s in normal operation, and **immediately** when the station's own detector sees a surprising cluster.

```json
{
  "v": 0,
  "station_id": "st-02",
  "time_source": "gnss",
  "window": ["2026-09-30T02:22:00Z", "2026-09-30T02:22:10Z"],
  "heard": [
    {"n": 305419896, "t": "2026-09-30T02:22:03.412Z", "seq": 1042, "batt_mv": 3280, "rssi": -112, "snr": -4.5}
  ],
  "missed": [
    {"n": 305419901, "expected": "2026-09-30T02:22:05Z", "last_heard": "2026-09-30T02:21:35.100Z", "consecutive": 2}
  ],
  "rejected_mic": 0
}
```

A binary or compressed encoding (e.g. CBOR) may replace JSON on expensive links such as satellite.

### Station health: `ofs/{deployment_id}/{station_id}/status`
Sent every 60 s. It includes battery, solar input, temperature, **noise floor per channel** (used to detect jamming), backhaul type, queue depth, firmware version and the number of nodes tracked, heard and missed.

Stations **buffer to flash** while the backhaul is down, then send the backlog in order when it returns.

## 3. Open data export

Arrivals and misses are exported as Parquet or CSV with the registry fields joined in (`lat`, `lon`, `elev`, `d`). The simulator reads and writes the same format. See [ARCHITECTURE.md §9](ARCHITECTURE.md#9-future-simulation).
