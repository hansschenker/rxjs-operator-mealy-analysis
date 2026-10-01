# `filter` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/filter.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `filter(predicate: (value, index) => boolean, thisArg?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. `thisArg` deprecated. |

SuperGrok is the main contributor of this analysis.

## Explanation

`filter` is a pipeable operator on the RxJS 7.x line. Stable. `thisArg` deprecated. Emit source nexts for which `predicate(value, index)` is true. Index increments on every source next, including filtered-out values. Error and complete pass through. Predicate throw is an error.

In plain terms, the operator keeps this memory: S = { active(i), stopped }. At subscription, before any source notification, that memory is active(0). It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2, 3, 4).pipe(filter(x => x % 2 === 0)) writes next(2) next(4) complete. Indexes seen by the predicate are 0, 1, 2, 3.

Details that a marble diagram often leaves out: Index is the source index, not the count of passed values. Does not unsubscribe early.

## Role in the notification machine

Emit source nexts for which `predicate(value, index)` is true. Index increments on every source next, including filtered-out values. Error and complete pass through. Predicate throw is an error.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active(i), stopped }`.

## 2. Initial state (S0)

`active(0)`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `active(i) × next → active(i+1)` if predicate returns.
- Throw or terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(v)` if predicate is true, else `ε`.
- Terminal inputs copy through.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { active(i), stopped }.

Memory at subscribe: active(0).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| active(i) × next | active(i+1) if predicate returns | next(v) if predicate is true, else ε |
| Throw or terminal | stopped | nothing named on a separate output row |
| Terminal inputs copy through | named by the output row; memory change is in the transition rows above | Terminal inputs copy through |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1, 2, 3, 4).pipe(filter(x => x % 2 === 0))` writes `next(2) next(4) complete`. Indexes seen by the predicate are 0, 1, 2, 3.

## Why this is Mealy rather than Moore

`G(active(i), next(v))` is either `next(v)` or `ε` based on predicate(state index, input value).

## Edge cases fixed by the 7.x source

- Index is the source index, not the count of passed values.
- Does not unsubscribe early.

## Source anchors

- `src/internal/operators/filter.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
