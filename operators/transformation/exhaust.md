# `exhaust` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/exhaust.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `exhaust(): OperatorFunction<ObservableInput<T>, T>` |
| Status on the 7.x line | Stable. Alias of `exhaustAll`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Source emits inners. If no inner is active, subscribe to the new inner and forward it. If an inner is active, drop the new inner without subscribing. Complete when the source is done and no inner is active.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, busy(inner), stopped }` plus `outerDone`.

## 2. Initial state (S0)

`idle`, outer not done.

## 3. Input alphabet (Z)

`{ outerNext(inner$), outerError, outerComplete, innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × outerNext → busy`.
- `busy × outerNext → busy` (dropped).
- `busy × innerComplete → idle`, then `stopped` if outer is done.
- Errors → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `outerNext → ε` in both idle and busy (subscribe action only when idle).
- `innerComplete → complete` if outer done, else `ε`.

## Worked trace

Clicks projected to 1-second inners: a click during an open inner produces no subscription and no output.

## Why this is Mealy rather than Moore

`outerNext` in `idle` vs `busy` is the same input symbol and different actions. The output word is `ε` either way, so the Mealy content is the subscription action in the output alphabet extension; the terminal `complete` decision still depends on state.

## Edge cases fixed by the 7.x source

- Dropped inners are never subscribed.
- Source file is a one-line alias to `exhaustAll`.

## Source anchors

- `src/internal/operators/exhaust.ts` re-exports `exhaustAll`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
