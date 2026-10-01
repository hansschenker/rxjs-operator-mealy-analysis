# `switchAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/switchAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `switchAll(): OperatorFunction<ObservableInput<T>, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe to each new inner and unsubscribe the previous one. Only the latest inner's values are forwarded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active inner | ⊥, outerDone, stopped }`.

## 2. Initial state (S0)

No inner.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` replaces active.
- `innerComplete` clears active.
- Outer done and idle → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- Previous inner's later signals are not delivered.

## Worked trace

Source emitting inner A then inner B unsubscribes A; only B's values appear after the switch.

## Why this is Mealy rather than Moore

Same machine as `switchMap` with identity project.

## Edge cases fixed by the 7.x source

- No queue.
- A sync inner can emit before the next outer next switches it.

## Source anchors

- `src/internal/operators/switchAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
