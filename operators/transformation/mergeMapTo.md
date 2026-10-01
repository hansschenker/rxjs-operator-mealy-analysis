# `mergeMapTo` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/mergeMapTo.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `mergeMapTo(inner, resultSelector?, concurrent?): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated. Use `mergeMap(() => inner)`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`mergeMapTo` is a pipeable higher-order operator on the RxJS 7.x line. Deprecated. Use `mergeMap(() => inner)`. `mergeMap` with a constant inner observable resubscribed for every accepted outer value.

In plain terms, the operator keeps this memory: Same as mergeMap. At subscription, before any source notification, that memory is Idle. It reacts to these events: Same higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 1).pipe(mergeMapTo(of('a'))) writes next('a') next('a') complete.

Details that a marble diagram often leaves out: Resubscribes; does not share one subscription across outer values. Deprecated.

## Role in the notification machine

`mergeMap` with a constant inner observable resubscribed for every accepted outer value.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Same as `mergeMap`.

## 2. Initial state (S0)

Idle.

## 3. Input alphabet (Z)

Same higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Same as `mergeMap`, project ignores the outer value.

## 6. Output function (G : S × Z → A*)

- Same as `mergeMap`.

## Worked trace

`of(1, 1).pipe(mergeMapTo(of('a')))` writes `next('a') next('a') complete`.

## Why this is Mealy rather than Moore

Same Mealy completion rule as `mergeMap`.

## Edge cases fixed by the 7.x source

- Resubscribes; does not share one subscription across outer values.
- Deprecated.

## Source anchors

- `src/internal/operators/mergeMapTo.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
