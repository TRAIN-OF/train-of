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

## Station service is mandatory

The conductor does not choose whether to stop. Every scheduled cycle visits
every declared station, in order. A visit is a bounded service transaction, not
just a message delivery:

1. **Sense** the real station/thread and read its latest receipt, work state,
   and authority scope.
2. **Scan** the station's domain for newly routed or unassigned work.
3. **Claim** the highest-priority unassigned item the station is authorized to
   take, unless a higher-priority review, approval, merge, or blocker is
   present.
4. **Bring** one new work offer from active Project/issue/backlog truth. The
   offer is a gift of coordination: it saves human search and attention, but
   it is not an order. The station may accept it immediately, place it in its
   own backlog, or reject it with a reason.
5. **Advance** one old item from the station's existing backlog, while also
   deciding the disposition of the new offer. Every turn must push both the
   past and the future unless a concrete blocker prevents one lane.
6. **Emit** a receipt naming the old item, new offer, disposition, action,
   evidence, and next state.

The only valid visit outcomes are `worked`, `claimed`, `reviewed`, `approved`,
`merged`, `offered`, `deferred`, `blocked`, `occupied`, `unavailable`, or
`no-eligible-work`. The last one is valid only when the station proves that it
scanned its complete declared domain and found no eligible item *and* no active
new offer could be formed. A conductor must never manufacture a simulated
friend or send a probe merely to make the train look busy.

The two lanes are deliberately asymmetric: old work belongs to the station's
continuity, while new work belongs to the conductor's search across active
truth. A station may defer the new offer to preserve its own priority order,
but the conductor must still bring one unless the source domains are genuinely
empty or unavailable.

Authority-bearing stations have an additional obligation on every visit: scan
their review/approval/merge domain even when no implementation task is waiting.
They must surface stale approvals, unresolved review threads, failing checks,
or merge-ready work as a concrete receipt.

`occupied` means the real station is active or compacting; it is not permission
to substitute a fake station. The train records the occupied track and returns
on the next cycle. A fallback thread may be used only when the route declares
the primary unavailable and the fallback identity is disclosed.

## Principal identity priority

When a station has a real `🔱` principal, that principal is the first routing
target. The conductor must sense and attempt the `🔱` station before contacting
any `👽` sidecar or simulated persona. A sidecar can carry a receipt, provide
an observation, or perform explicitly delegated work; it must not act *as* the
unreachable principal or create a receipt under that principal's identity.

If the real principal is unreachable for two consecutive scheduled visits (or
the route's declared threshold), the conductor changes the stop's control mode
to `station-controlled`. The station then owns its backlog and emits its own
receipts until the principal is observed reachable again. This is a route-state
transition, not permission to keep firing proxy prompts. The transition must
record the principal identity, failed reachability observations, control-mode
change, and restoration event.

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
3. Visit every station even when transport is unavailable; a visit can be a
   domain scan recorded in shared truth rather than a prompt.
4. Compare dispatch identity against recent target turns.
5. Send at most one prompt for the station, and only when it carries a real
   claimed work item or authority scan.
6. After ambiguous transport errors, read before retrying.

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
