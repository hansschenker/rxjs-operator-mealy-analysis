# `exhaustMap` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/exhaustMap.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `exhaustMap(project, resultSelector?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Project outer values to inners, but only subscribe when no inner is active. Outer values that arrive while busy are ignored, not queued. This is `exhaust` plus a projection.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, busy, stopped }` with `outerDone`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

Higher-order alphabet; `outerNext(v)` carries the value passed to `project`.

## 4. Output alphabet (A)

`{ next(r), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × outerNext → busy` if project returns.
- `busy × outerNext → busy` with no subscribe.
- `innerComplete → idle` or `stopped` if outer done.
- Project throw or inner error → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `outerNext → ε`.
- `innerComplete → complete` only when outer is done and state returns to idle.

## Worked trace

`clicks.pipe(exhaustMap(() => interval(1000).pipe(take(3))))` ignores clicks until the current three ticks finish.

## Why this is Mealy rather than Moore

Acceptance of `outerNext` is a state predicate. The emitted letters come from inner inputs while `busy`.

## Edge cases fixed by the 7.x source

- No queue. Contrast `concatMap`, which would buffer the ignored clicks.
- Result selector can still see the outer value that was accepted.

## Source anchors

- `src/internal/operators/exhaustMap.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
