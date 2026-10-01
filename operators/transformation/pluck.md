# `pluck` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/pluck.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `pluck(...properties): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated. Use `map(x => x.a.b)`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Project each next through a path of property names. Missing segments yield `undefined`. Error and complete pass through.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, stopped }`. No index.

## 2. Initial state (S0)

`active`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(path(v)), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `active × next → active`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `active × next(v) → next(v[k1][k2]...)`.
- Terminal inputs copy through.

## Worked trace

`pluck('a', 'b')` on `{a:{b:1}}` writes `next(1)`. On `{}` writes `next(undefined)`.

## Why this is Mealy rather than Moore

Output letter is the path applied to the input value. State only gates whether the machine is still active.

## Edge cases fixed by the 7.x source

- Does not throw on missing properties; it emits `undefined`.
- Deprecated.

## Source anchors

- `src/internal/operators/pluck.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
