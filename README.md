# TRAIN-OF/train-of

Class definition and SDK layer for `TRAIN-OF/*` repositories.

`TRAIN-OF/*` repos describe recurring agentic routes: ordered stations, cadence,
handoff rules, conductor behavior, telemetry semantics, and the thin event pulses
that wake trains without embedding stale policy in automation prompts.

## Protocol version

This repository defines `train-of.route.v2`, SemVer `2.0.0`. This is a
breaking change: a station may not register on a route without a verified
remote address and application-level ACK. Each station must publish its host,
thread, machine, account, cwd, reply-to address, verification time, and
transport evidence (nonce and ACK time). A local sidecar is a fallback, never
a substitute for an unavailable principal.

Older `v0` documents are not wire-compatible with v2. Migrate them by
collecting and verifying every station address before publishing the route.

## Why this exists

The first live train experiments showed a useful split:

- a **slow wave**: scheduled train pulses, station receipts, Project/PR routing;
- a **lightning flash**: direct cross-thread/cross-machine messages for disclosure,
  blockers, role sync, and handoffs that should not wait for the next route cycle.

This repo keeps that design reusable across train lines instead of letting each
automation prompt become a hidden fork of the operating manual.

## Core doctrine

1. **Arrival events should be thin.** A heartbeat should say that a train arrived;
   it should not carry the whole route doctrine.
2. **State may ride with the train.** Conductor-driven lines can attach a compact
   loadout/cargo path to the train identity for that arrival.
3. **Doctrine belongs in shared truth.** Station order, dispatch rules, and
   escalation paths should live in repos, Project items, issues, or signed receipts.
4. **Cross-thread direct messaging is lightning, not the train.** Use it for
   orientation, blockers, and explicit handoffs. Preserve identity boundaries.
5. **No fake progress.** A train may report no-change, occupied track, stale
   telemetry, or unavailable station; those are valid outcomes.

## Minimal train pulse

```text
🚂🟥 tick
```

The conductor resolves current station, cargo, authority, and routing from shared
truth at runtime.

## Suggested repo shape

```text
README.md
doctrine/
  train-system.md
schemas/
  train-route.schema.json
examples/
  red-line.route.json
```

## Relationship to station repos

`TRAIN-OF/*` owns route behavior. `STATION-OF/*` owns station behavior. A train
may visit a station role shared by multiple distinct agentic thread identities,
but it must not cause one thread to impersonate another thread's haecceity/card.
