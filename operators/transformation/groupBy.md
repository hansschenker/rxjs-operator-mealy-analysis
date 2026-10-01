# `groupBy` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/groupBy.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `groupBy(keySelector, elementSelector?, durationSelector?, connector?): OperatorFunction<T, GroupedObservable<K, R>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`groupBy` is a pipeable operator on the RxJS 7.x line. Stable. Map each source value to a key. Emit a `GroupedObservable` the first time a key is seen. Later values for that key are nexted into the subject's group. `durationSelector`, if present, closes a group when its notifier emits. Source complete completes every open group and then the outer. Each group is its own small machine.

In plain terms, the operator keeps this memory: S = { map key → { subject, open }, stopped }. Infinite key space possible. At subscription, before any source notification, that memory is Empty map. It reacts to these events: { next(v), error, complete, durationNext(k), durationError(k), unsubscribe } plus group-subscriber subscribe/unsubscribe (refcounts on the connector subject).

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Values {id:1}, {id:1}, {id:2} emit two group observables. The first group writes two elements; the second writes one.

Details that a marble diagram often leaves out: Late subscribers to a group see only what the connector subject replays (`Subject` by default, so nothing already past). Duration complete also closes the group.

## Role in the notification machine

Map each source value to a key. Emit a `GroupedObservable` the first time a key is seen. Later values for that key are nexted into the subject's group. `durationSelector`, if present, closes a group when its notifier emits. Source complete completes every open group and then the outer. Each group is its own small machine.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { map key → { subject, open }, stopped }`. Infinite key space possible.

## 2. Initial state (S0)

Empty map.

## 3. Input alphabet (Z)

`{ next(v), error, complete, durationNext(k), durationError(k), unsubscribe }` plus group-subscriber subscribe/unsubscribe (refcounts on the connector subject).

## 4. Output alphabet (A)

`{ next(GroupedObservable), error, complete }` on the outer, and `{ next(element), error, complete }` on each group.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` computes key. Missing key adds a group. Existing open key stays.
- `durationNext(k)` marks that group closed so a later value with the same key opens a new group.
- Source complete closes all groups and stops.
- Source error errors all groups and stops.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(group$)` if the key is new, else `ε` on the outer. The group machine writes `next(elementSelector(v))`.
- `complete →` complete each group, then outer `complete`.
- `error →` error each group and the outer.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { map key → { subject, open }, stopped }. Infinite key space possible.

Memory at subscribe: Empty map.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next(v) computes key. Missing key adds a group. Existing open key stays | next(v) computes key. Missing key adds a group. Existing open key stays | next(group$) if the key is new, else ε on the outer. The group machine writes next(elementSelector(v)) |
| durationNext(k) marks that group closed so a later value with the same key opens a new group | durationNext(k) marks that group closed so a later value with the same key opens a new group | nothing named on a separate output row |
| Source complete closes all groups and stops | Source complete closes all groups and stops | complete each group, then outer complete |
| Source error errors all groups and stops | Source error errors all groups and stops | error each group and the outer |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Values `{id:1}`, `{id:1}`, `{id:2}` emit two group observables. The first group writes two elements; the second writes one.

## Why this is Mealy rather than Moore

Outer `G` on source next is either a new group or `ε` depending on key memory. Group `G` writes the element. Both are input-and-state functions.

## Edge cases fixed by the 7.x source

- Late subscribers to a group see only what the connector subject replays (`Subject` by default, so nothing already past).
- Duration complete also closes the group.

## Source anchors

- `src/internal/operators/groupBy.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
