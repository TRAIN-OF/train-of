# TRAIN-OF/train-of

Class definition and SDK layer for `TRAIN-OF/*` repositories.

`TRAIN-OF/*` repos describe recurring agentic routes: ordered stations, cadence,
handoff rules, conductor behavior, telemetry semantics, and thin event pulses.
Concrete train repositories own route cursor and cargo state; station registries
own endpoint authority; adapters own harness-specific delivery.

## Protocol version

This repository defines `train-of.route.v2`, SemVer `2.1.0`. v2 remains the
breaking station-registration boundary: a station may not register without a
verified remote address and application-level ACK. Each station publishes its
host, thread, machine, account, cwd, reply-to address, verification time, and
transport evidence.

Version 2.1.0 adds optional conditional event arrivals. It is backward
compatible: existing v2.0 routes continue to use scheduled arrivals only.
Conditional arrivals rehydrate authoritative GitHub state for one event and
must never advance or reset a concrete route cursor.

## Core doctrine

1. **Scheduled arrivals advance the train.** They resolve cargo, do bounded
   work, verify monitor health, and persist next-arrival state.
2. **Conditional arrivals answer one event.** They use a deduplicated,
   authenticated packet and leave the scheduled route intact.
3. **Arrival events stay thin.** Payloads reference durable state rather than
   carrying route doctrine.
4. **Doctrine belongs in shared truth.** Train behavior is class-level; route
   state is concrete and never belongs in this repository.
5. **No fake progress.** Occupied track, stale telemetry, and named blockers
   are valid outcomes.

See [`doctrine/train-system.md`](doctrine/train-system.md) and
[`schemas/train-route.schema.json`](schemas/train-route.schema.json).

See [`doctrine/line-classes.md`](doctrine/line-classes.md) for the shared
orange express and blue deep-survey route classes, including their station
depth policies and foreign-repository-key convention.
