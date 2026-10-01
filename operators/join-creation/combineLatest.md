# `combineLatest` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/combineLatest.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `combineLatest(sources, resultSelector?): Observable<T[]>` |
| Status on the 7.x line | Stable as a creation function. The pipeable form on 7.x is `combineLatestWith` / `combineLatestAll`. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe to every source. Remember the latest value of each. Emit an array (or projected tuple) only once every source has produced at least one value, and again whenever any source emits after that. Complete when every source has completed. Error if any source errors. A source that completes without a value prevents any emission and completes the output when all have settled without a full tuple.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = records of { latest: V | ⊥, done: bool } per source, plus `stopped`. Infinite because values are arbitrary. Control flag `ready = ∀ latest ≠ ⊥`.

## 2. Initial state (S0)

All `latest = ⊥`, all `done = false`.

## 3. Input alphabet (Z)

`{ subscribe, next_i(v), error_i(e), complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next_i(v)` stores `latest_i = v`.
- `complete_i` sets `done_i`. If all done, → `stopped`.
- `error_i → stopped`.
- `unsubscribe → stopped` and unsubscribe all.

## 6. Output function (G : S × Z → A*)

- `next_i(v) → next(snapshot)` if every source has a latest, else `ε`.
- `error_i(e) → error(e)`.
- `complete_i → complete` if all sources are done, else `ε`. No extra next on complete.

## Worked trace

Sources `A: 1, 2` and `B: a`. After `1` the word is `ε` (B missing). After `a` the word is `next([1, a])`. After `2` the word is `next([2, a])`.

## Why this is Mealy rather than Moore

`G` needs both the stored latests and the arriving `next_i` to decide between `ε` and `next(snapshot)`. That is Mealy.

## Edge cases fixed by the 7.x source

- Dictionary form uses keys instead of indexes; the tuple shape changes, the machine does not.
- Empty source list completes immediately.
- Completion of a source that already has a latest does not emit by itself.

## Source anchors

- Creation entry: `src/internal/observable/combineLatest.ts`.
- Operator implementation shared with `src/internal/operators/combineLatest.ts` / `combineLatestAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
