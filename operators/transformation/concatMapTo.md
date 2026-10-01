# `concatMapTo` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/concatMapTo.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `concatMapTo(innerObservable, resultSelector?): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated in 7.x. Use `concatMap(() => inner)`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Identical machine to `concatMap` except `project` ignores the outer value and returns the same `ObservableInput` each time. The input is still subscribed per outer value, not shared.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Same as `concatMap`: `{ active, queue, outerDone, stopped }`.

## 2. Initial state (S0)

Idle, empty queue.

## 3. Input alphabet (Z)

Same higher-order alphabet as `concatMap`.

## 4. Output alphabet (A)

`{ next(r), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Same transitions as `concatMap`. The constant inner is resubscribed for every dequeued outer value.

## 6. Output function (G : S × Z → A*)

- Same output rules as `concatMap`. The outer value is not part of the output unless a deprecated result selector closes over it.

## Worked trace

`of(1, 2).pipe(concatMapTo(of('x')))` writes `next('x') next('x') complete`.

## Why this is Mealy rather than Moore

Same Mealy shape as `concatMap`. Deprecation does not change the tuple.

## Edge cases fixed by the 7.x source

- The same observable object is resubscribed; it is not merged concurrently.
- Cold inners rerun. A hot inner would share its timeline per subscription rules.

## Source anchors

- `src/internal/operators/concatMapTo.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
