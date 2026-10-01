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
