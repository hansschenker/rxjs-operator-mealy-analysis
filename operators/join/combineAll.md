# `combineAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Deprecated alias |
| RxJS 7.x source | `src/internal/operators/combineAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `combineAll(project?): OperatorFunction<ObservableInput<T>, T[]>` |
| Status on the 7.x line | Deprecated alias of `combineLatestAll`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

`combineAll` is a one-line re-export of `combineLatestAll`. The machine is the higher-order combineLatest machine: collect inners until the outer completes, then emit snapshots once every inner has a latest.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Same as `combineLatestAll`: collected inners, latest per inner, outerDone, stopped.

## 2. Initial state (S0)

No inners collected.

## 3. Input alphabet (Z)

`{ outerNext(inner$), outerComplete, outerError, innerNext_i, innerError_i, innerComplete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Identical to `combineLatestAll`.
- Deprecation does not add a state.

## 6. Output function (G : S × Z → A*)

- Identical to `combineLatestAll`: an inner next writes a snapshot only when every collected inner has a latest.

## Worked trace

Same word as `combineLatestAll` on the same higher-order source.

## Why this is Mealy rather than Moore

The alias does not change the Mealy decision: emit-or-ε depends on readiness state and the arriving inner next.

## Edge cases fixed by the 7.x source

- File is a re-export. Behavior lives in `combineLatestAll.ts`.
- Removed in later majors.

## Source anchors

- `src/internal/operators/combineAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
