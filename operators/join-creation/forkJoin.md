# `forkJoin` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/forkJoin.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `forkJoin(sources): Observable<T[] | Record<K, V>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe to all sources. Keep only the last value of each. Emit once, when every source has completed, the array or dictionary of last values, then complete. If any source errors, error and unsubscribe the rest. If any source completes without a value, complete without emitting.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = per-source { last: V | ⊥, done: bool } plus `stopped`.

## 2. Initial state (S0)

All `last = ⊥`, `done = false`.

## 3. Input alphabet (Z)

`{ subscribe, next_i(v), error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(lasts), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next_i(v)` overwrites `last_i`.
- `complete_i` sets `done_i`. All done → `stopped`.
- `error_i → stopped`.

## 6. Output function (G : S × Z → A*)

- `next_i → ε` always (forkJoin does not emit on next).
- `complete_i → next(lasts) · complete` if every source is done and every source has a last value.
- `complete_i → complete` if every source is done but some last is ⊥.
- `error_i(e) → error(e)`.

## Worked trace

`forkJoin([of(1, 2), of('a')])` writes a single `next([2, 'a']) complete` after both complete. The `1` is overwritten and never emitted.

## Why this is Mealy rather than Moore

The complete input is what triggers the output word, and the word's payload is the state. Classic Mealy: completion is the input, the tuple is memory.

## Edge cases fixed by the 7.x source

- Empty argument list completes without a next.
- Dictionary keys are preserved.
- Unlike `combineLatest`, intermediate snapshots are not outputs.

## Source anchors

- `src/internal/observable/forkJoin.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
