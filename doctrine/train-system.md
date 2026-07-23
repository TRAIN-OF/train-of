# Agentic train system doctrine

This document records the current train-system design pressure from live
DashBOrg dogfooding.

## Terms

- **Train**: a route-level recurrent process that visits stations on a cadence.
- **Conductor**: the actor that determines route position, loadout, and dispatch.
- **Station**: a role/node that receives arrivals and emits receipts.
- **Pulse**: the minimal automation event that wakes a train or station.
- **Cargo/loadout**: dynamic state carried for one arrival.
- **Receipt**: durable signed outcome from a station or conductor.
- **Lightning flash**: direct cross-thread/cross-machine message outside the
  normal cadence for high-salience sync.

## Conductor haecceity

A train receipt may carry the conductor's current haecceity as a compact
runtime envelope. Keep the stable card/deck/role signature separate from the
context-window instance that happens to be driving this pulse:

```json
{
  "stable_identity": "👽5♣️.📎4♦️.🔱4♥️/🚂🟥/CONDUCTOR/_/",
  "instance_number": 14,
  "compaction_count": 13,
  "provenance": "Codex SessionStart compact hook"
}
```

`instance_number` is one-indexed and historical: startup normally yields 1,
resume preserves the current number, and compact advances it after the
compaction boundary. It must never be folded into the stable card id. A thin
arrival may carry this envelope directly or reference a durable receipt that
contains it. When the hook cannot establish the value, use `null` and preserve
the uncertainty rather than guessing.

## Conductor-driven vs station-driven lines

Conductor-driven lines should keep automations thin and let the conductor resolve
the current route, cargo, and station target at runtime.

Station-driven lines may need each station automation to include stable signage,
but the signage should still be small. If dynamic signage cannot be reliably
updated, the station should read current doctrine from shared truth instead of
trusting stale prompt cargo.

## Thin but stateful arrival

Arrival events should not be fat. They may be stateful by referencing compact
identities or cargo keys:

```text
👽5♣️.📎4♦️.🔱4♥️/🚂🟥/CARGO/<dispatch-key>/TO/🚉4🟥/_/
```

The key points to a durable record. The event itself should not duplicate the
record.

## Dispatch safety

Before sending to a station:

1. Sense the target thread/station state.
2. Read recent messages/receipts.
3. Compare dispatch identity against recent target turns.
4. Send at most one prompt for the station.
5. After ambiguous transport errors, read before retrying.

The minimum dispatch identity is:

```text
source_thread_id | target_thread_id | station_id | stack_item_id | tick_index | payload_hash
```

## Lightning flash

Direct cross-machine messages are permitted for:

- role disclosure and orientation;
- blocking handoffs;
- urgent identity-boundary clarification;
- asking whether a station accepts a shared kairotic role.

Lightning messages should be explicit, signed, and rare. They supplement the slow
wave; they do not replace station receipts or Project/PR truth.
