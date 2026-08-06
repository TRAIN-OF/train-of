# Route classes

Route classes describe the quality of a train visit. They are not identities
of the train, station, conductor, or cargo. The class can be layered onto any
train card and any station sequence.

## 🟧 Orange Line — Express

`line_class.id`: `express`

- one pass through the ten stations;
- one bounded health or existence signal per station;
- thin receipts only;
- no foreign-repository expansion;
- no mutation unless a separate action explicitly authorizes it.

The orange line answers: **where is the route healthy, stale, or blocked?**

## 🟦 Blue Line — Deep Survey Loop

`line_class.id`: `deep-survey`

The blue line is a hub-and-return route. It begins at `models-of/`, visits
the concrete registries and runtime boundaries, and returns to `models-of/`
for reconciliation:

1. models — define the expected abstraction and relations;
2. cards — verify owned identities and provenance;
3. decks — verify collections and possession semantics;
4. bestiary — verify permutation and transformation identity;
5. maps — verify concrete arrangements and coordinates;
6. agent continuity — verify sessions, assignments, and receipts;
7. privacy — verify public/private evidence boundaries;
8. deployment — verify staging, devline, and browser handoff;
9. performance — run bounded realistic checks;
10. atomic history — verify commits, branches, and promotion gates;
11. models — reconcile the abstraction against observed evidence.

The blue line varies depth per station. Healthy registries may receive a
shallow schema check; contradictory or high-risk stations receive source,
cross-reference, provenance, and runtime checks. Every deep arrival must name
its evidence, missing records, contradictions, and next action.

## Foreign repository keys

Route records may reference concrete cargo without copying it. Use a stable
`foreign_repo_key` such as:

```text
CARDS-OF/qadence-tessel-00q
DECKS-OF/qadence-tessel-00q
MAPS-OF/qadence-tessel-00q
BESTIARY-OF/TDOTDH
MODELS-OF/qadence-tessel-00q
```

These are opaque references. A key does not claim that the repository exists,
is public, or is currently registered; the receiving station must resolve and
report its status. In particular, the Qadence `MODELS-OF` key is currently a
planned reference, not an invented repository.

## Route-class payload

The optional route declaration is intentionally compact:

```json
{
  "line_class": {
    "id": "deep-survey",
    "color": "🟦",
    "loop_shape": "hub-and-return",
    "station_depth_policy": "vary-by-risk",
    "foreign_repo_keys": ["CARDS-OF/qadence-tessel-00q", "MAPS-OF/qadence-tessel-00q"]
  }
}
```

The color is a team-color overlay, not a new deck or card identity.
