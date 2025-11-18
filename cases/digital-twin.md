# Digital Twin Platforms (Industrial/IoT): Dual-Native Case

Domain: Industrial IoT / Digital Twins

- Human Representation (HR): 3D visualizations, dashboards, and control panels for assets, lines, and facilities.
- Machine Representation (MR): Telemetry streams (e.g., MQTT, OPC UA), historian/query APIs, and twin state APIs.
- Shared Data: Asset models, sensor readings, states, alarms, configurations.

## Current State
- Discovery: Asset/twin registries exist but vary; streams are discovered via broker topics; no standard index pairing HR and MR.
- Synchronization/Validators: Timestamps, sequence numbers, twin versions; adequate CIDs in principle but not unified.
- Linking: UI rarely exposes direct data endpoints; MR often lacks links back to the exact HR view.

## Pattern Gaps
- Missing consistent HR <-> MR links between specific HR views and their MR topics/APIs.
- No consolidated Resource Index mapping assets to (hr, mr, cid) for telemetry and state.
- Equivalence testing between displayed readings and machine data is ad-hoc.

## Recommended Upgrades
1) Bidirectional linking
   - HR: From a selected asset/sensor, expose links to MR (topic names, API endpoints) for real-time and historical data.
   - MR: Include an `hr_url` back to the exact twin/asset page in metadata or registry records.
2) Shared CID (content identity)
   - Use twin version or timestamp+sequence for readings shown in HR; surface the same CID in MR responses.
3) Standardized discovery (Resource Index)
   - Publish a catalog enumerating assets with (rid, hr, mr endpoints, cid, updatedAt); support incremental updates and partitioning by area/line.

## Expected Benefits
- Faster troubleshooting and analytics by connecting visuals to raw machine data.
- Deterministic reconciliation between what operators see and what algorithms consume.
- Lower polling costs via zero-fetch checks on twin versions and last-seen positions.

## Risks/Considerations
- Control-plane links must be access-controlled; avoid accidental exposure of privileged operations.
- Document CID semantics for streams (position over payload hash) to prevent misuse.

