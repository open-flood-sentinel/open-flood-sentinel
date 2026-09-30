# Contributing to Open Flood Sentinel

Thank you for helping. Every contribution counts, whether it is code, a schematic, a correction, a translation or a field report.

## Ways to contribute

| You are… | Start here |
|---|---|
| New to the project | Read the [README](README.md), then pick an issue labelled `good first issue` |
| Embedded developer | `firmware/`, issues labelled `firmware` |
| Hardware designer | `hardware/`, issues labelled `hardware` |
| Backend / data developer | `base-station/`, `simulation/` |
| Hydrologist / civil engineer | Review [ALGORITHMS.md](docs/ALGORITHMS.md), issues labelled `hydrology` |
| Someone who lives in a flood-prone area | Share local knowledge in Discussions → *Field & Community* |
| Writer / translator | Issues labelled `docs` or `translation` |

## How to contribute

1. **Discuss first for anything big.** Open an issue or a Discussion before starting large features, protocol changes or hardware redesigns, so that nobody's work is wasted.
2. **Fork** the repo and create a branch: `feat/node-sleep`, `fix/station-dedup`, `docs/hardware-bom`.
3. Keep pull requests **small and focused**. Say what you changed and how you tested it.
4. Link the issue in your PR (`Closes #12`).
5. A maintainer will review your PR. Be patient and kind; we are volunteers.

## Standards

- **Firmware**: say which board you tested on. Include measured current draw when you change power behaviour.
- **Hardware**: include source files (KiCad preferred) as well as exports (PDF schematic, Gerbers, BOM with part numbers).
- **Software**: include tests where practical. Estimation logic **must** have tests, ideally driven by the simulator.
- **Protocol changes**: update [PROTOCOL.md](docs/PROTOCOL.md) and bump the version.
- **Units**: SI units throughout, times in UTC.

## Safety-critical changes

Changes to **alert logic, thresholds, the protocol or heartbeat timing** affect whether people get warned. Such PRs:
- need review from **2 maintainers**
- must include tests or simulation results
- must explain the failure modes considered

## Commit messages

Use a short imperative summary: `Add heartbeat channel hopping`, `Fix loss time when station reconnects`.

## Code of Conduct

By taking part, you agree to follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Licensing of contributions

By contributing, you agree that your contributions are licensed under the project licenses listed in the README (Apache-2.0 for software, CERN-OHL-S-2.0 for hardware, CC BY-SA 4.0 for docs).
