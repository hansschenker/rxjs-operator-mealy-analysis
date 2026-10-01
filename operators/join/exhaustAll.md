# `exhaustAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/exhaustAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `exhaustAll(): OperatorFunction<ObservableInput<T>, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe to an inner only if none is active. Inners that arrive while busy are dropped, not queued. `exhaust` is this function.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, busy, outerDone, stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × outerNext → busy`.
- `busy × outerNext → busy` (dropped).
- `innerComplete → idle` or `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `outerNext → ε`.
- Complete when outer is done and idle.

## Worked trace

A second inner emitted before the first completes never gets a subscription.

## Why this is Mealy rather than Moore

Drop vs subscribe is a transition on the same `outerNext` symbol, gated by busy state.

## Edge cases fixed by the 7.x source

- Dropped inners are not subscribed.
- See `exhaust.ts`, which re-exports this.

## Source anchors

- `src/internal/operators/exhaustAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
