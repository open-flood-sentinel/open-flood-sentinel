# Roadmap

Each phase has a clear **exit criterion**. Progress is tracked on the GitHub Project board.

## Phase 0: Foundation *(now)*
- [x] Concept and initial documentation
- [ ] License chosen and added
- [ ] GitHub org, Discussions, labels and project board set up
- [ ] Community chat opened
- [ ] First 10–15 `good first issue`s published
- [ ] Protocol v0 reviewed by at least 2 contributors

**Exit:** Contributors can find out what to work on without asking.

## Phase 1: Bench prototype
- [ ] Node firmware: sleep, jittered heartbeat with channel hopping, signed packets
- [ ] Station firmware: heartbeat-due tracking, detecting missed heartbeats
- [ ] Station firmware: LoRa RX, timestamping, MQTT uplink, store-and-forward, siren output
- [ ] Base: ingestion, node-state tracking, alert levels, a simple dashboard
- [ ] **Simulator**: thousands of virtual nodes, random background failures, collisions and synthetic floods, run through the real base software
- [ ] Measured node sleep current and projected battery life

**Exit:** In simulation, floods are detected with correct speed, level and ETA, while random failures, station outages and jamming cause no false alarms. On the bench, pulling power from a group of about 20 real nodes in sequence produces the expected pattern.

## Phase 2: Field test (no public alerts)
- [ ] Waterproof enclosure and breakaway mount designs
- [ ] Solar + LiFePO4 station build, including a cold-weather BMS
- [ ] Range tests in real valley terrain
- [ ] Deploy about 100–300 nodes at a small seasonal stream, with manual detach drills in sequence
- [ ] Measure real background failure rate, packet loss and battery drain
- [ ] Deployment registry tooling (survey positions and heights)

**Exit:** 3 months of unattended operation with over 95% heartbeat delivery and every drill detected.

## Phase 3: Shadow pilot with a partner community
- [ ] Partnership with a local authority or NGO and a community
- [ ] Alerts go to operators and responders only
- [ ] One full flood or monsoon season logged
- [ ] Post-season review: detections, misses, false alarms

**Exit:** The safety checklist in [SAFETY.md](SAFETY.md) is complete.

## Phase 4: Community alerting
- [ ] Public alert channels (sirens, SMS, loudspeakers) agreed with authorities
- [ ] Build and deployment guides in local languages
- [ ] Replicable kit, so other communities can deploy it themselves

## Phase 5: Simulation and modelling
- [ ] Open dataset of events from deployments
- [ ] Flood visualisation from real event data and terrain
- [ ] Algorithm calibration against observed floods, with discharge estimates
