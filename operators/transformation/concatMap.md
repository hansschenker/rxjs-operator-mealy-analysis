# `concatMap` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/concatMap.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `concatMap(project, resultSelector?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Project each source value to an inner observable and flatten with concurrency 1. Later source values wait in a queue until the active inner completes. Inner and outer errors fail the output. Complete when the outer is done and the queue and active inner are empty.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active inner | ⊥, queue of outer values, outerDone, stopped }`.

## 2. Initial state (S0)

No active inner, empty queue, outer not done.

## 3. Input alphabet (Z)

`{ outerNext(v), outerError, outerComplete, innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(r), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` enqueues, and starts an inner if none is active.
- `innerNext` leaves the queue unchanged.
- `innerComplete` pops the next queued outer into `active`, or clears `active`.
- Both outer done and idle → `stopped`.
- Any error → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext(r) → next(r)` (after optional result selector).
- `innerComplete → complete` only if outer is done and the queue is empty, else `ε`.
- `outerNext → ε`.
- Errors copy through.

## Worked trace

Outer `1, 2` with project `x => of(x, x)` writes `next(1) next(1) next(2) next(2)`. The second inner cannot start before the first completes.

## Why this is Mealy rather than Moore

`innerComplete` writes `complete` or `ε` based on queue and outerDone state. The input is the same symbol; the word differs. That is Mealy.

## Edge cases fixed by the 7.x source

- Concurrency is fixed at 1. That is the only difference from `mergeMap` in `mergeInternals`.
- Project throw → error, queue discarded.

## Source anchors

- `src/internal/operators/concatMap.ts` calls `mergeInternals` with `concurrent = 1`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
