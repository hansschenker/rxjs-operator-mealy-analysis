# `switchScan` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order accumulator |
| RxJS 7.x source | `src/internal/operators/switchScan.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `switchScan(accumulator, seed): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Like `mergeScan` with switch semantics. A new outer value unsubscribes the active accumulator inner and subscribes to `accumulator(latestAcc, value)`. Inner emissions update `acc` and are forwarded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { acc, active inner | ⊥, outerDone, stopped }`.

## 2. Initial state (S0)

`acc = seed`, no inner.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next(r), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` unsubscribes the active inner and subscribes to the new accumulation.
- `innerNext` replaces `acc`.
- `innerComplete` clears active; if outer done → `stopped`.
- Errors → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext(r) → next(r)`.
- `outerNext → ε` (switch action only).
- `innerComplete → complete` if outer done, else `ε`.

## Worked trace

A fast outer with a slow accumulator inner: only the latest accumulation survives; previous inner nexts stop.

## Why this is Mealy rather than Moore

Switch is a transition on outer next that depends on there being an active inner. Emissions still come from inner inputs applied to acc state.

## Edge cases fixed by the 7.x source

- Seed is required and not emitted up front.
- Unsubscribed inner emissions are not outputs.

## Source anchors

- `src/internal/operators/switchScan.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
