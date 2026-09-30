# Safety and responsible deployment

This project exists to save lives. A warning system that fails silently, or cries wolf until people stop listening, can cost lives. Please read this before you deploy anything.

## Principles

1. **Supplement, never replace.** Official warnings, local knowledge and people's own judgement still come first. Never tell a community to rely on this system alone.
2. **Work with authorities.** Involve your national or local disaster management authority, local government and emergency services before you run any public alerts. Warnings work best when they fit into an existing response plan.
3. **The community owns it.** People living downstream must understand what the alerts mean, what to do when they hear one, and where to go. **A warning without an evacuation plan saves no one.**
4. **Be honest about limits.** Publish coverage, known blind spots, system health and false-alarm rates.
5. **Test before trusting.** Test on the bench, then run field drills, then run a shadow pilot (alerts go to operators only) before any public alerting. See [ROADMAP.md](ROADMAP.md).
6. **Guard against alert fatigue.** Every false public alarm makes people trust the next one less. Use corroboration rules, and review every false alarm openly.
7. **Visible health.** A system that is down must *look* down to operators. Monitor heartbeats at every tier.
8. **Maintenance is a promise.** Deployment is not a one-off event. Schedule battery checks, replacement nodes after floods, and pre-season inspections.
9. **Field safety.** Never install or retrieve nodes during high flow, heavy rain or flood warnings. Work in pairs.
10. **Respect people and land.** Get permission from landowners and communities. Do not collect personal data you don't need.

## Before any public alerting (checklist)

- [ ] Agreement with the local disaster management authority or local government
- [ ] Alert thresholds and escalation rules agreed and documented
- [ ] Evacuation routes and safe areas known to the community
- [ ] Community briefed in their own language, with drills done
- [ ] At least one full monsoon or flood season of shadow operation
- [ ] Every tier has a monitored heartbeat, and an operator on-call rota exists
- [ ] False-alarm review process in place
- [ ] Maintenance schedule and funding in place

## Disclaimer

This software and hardware are provided **as is, without warranty of any kind**. The authors and contributors accept no liability for any failure to warn, false warnings, or damage arising from their use.
