# `zip` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/zip.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `zip(...sources, resultSelector?): Observable<T[]>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Pair values by index. Each source has a queue. When every queue is non-empty, shift one value from each and emit the tuple. Complete when any source completes and its queue cannot form another tuple. Error on any error.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = per-source queue (sequence of V) plus done flags, or `stopped`. Queues make `S` infinite.

## 2. Initial state (S0)

Empty queues, no source done.

## 3. Input alphabet (Z)

`{ subscribe, next_i(v), error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next_i(v)` appends to queue i. If all queues are non-empty, shift one from each.
- `complete_i` marks done. If queue i is empty and done, → `stopped`.
- `error_i → stopped`.

## 6. Output function (G : S × Z → A*)

- `next_i → next(tuple)` if the append filled the last empty queue, else `ε`.
- `complete_i → complete` if no further tuple can be formed, else `ε`.
- `error_i(e) → error(e)`.

## Worked trace

`zip(of(1, 2), of('a'))` writes `next([1, 'a']) complete`. The `2` stays queued and is dropped when the shorter source completes.

## Why this is Mealy rather than Moore

Emit-or-not is a function of the queues (state) and the arriving next (input). Index alignment is the state invariant.

## Edge cases fixed by the 7.x source

- Unlike `combineLatest`, values are consumed, not reused.
- A result selector projects the tuple inside `G` only.

## Source anchors

- `src/internal/observable/zip.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
