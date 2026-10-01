# `switchMapTo` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/switchMapTo.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `switchMapTo(inner, resultSelector?): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated. Use `switchMap(() => inner)`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

`switchMap` whose project ignores the outer value and resubscribes to the same inner observable.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Same as `switchMap`.

## 2. Initial state (S0)

No inner.

## 3. Input alphabet (Z)

Same higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Same as `switchMap`.

## 6. Output function (G : S × Z → A*)

- Same as `switchMap`.

## Worked trace

`clicks.pipe(switchMapTo(interval(1000)))` restarts the interval on every click; only the latest interval emits.

## Why this is Mealy rather than Moore

Same switch Mealy machine as `switchMap`.

## Edge cases fixed by the 7.x source

- Resubscribe, not a shared subscription.
- Deprecated.

## Source anchors

- `src/internal/operators/switchMapTo.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
