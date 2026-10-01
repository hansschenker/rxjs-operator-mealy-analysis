# `map` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/map.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `map(project: (value, index) => R, thisArg?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. `thisArg` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

On each source next, call `project(value, index)` and emit the result. Index starts at 0 and increments per source next. Error and complete pass through. A throw from `project` becomes an error notification.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active(i) | i ∈ ℕ } ∪ { stopped }`.

## 2. Initial state (S0)

`active(0)`.

## 3. Input alphabet (Z)

`{ next(v), error(e), complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(r), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `active(i) × next(v) → active(i+1)` if project returns.
- `active(i) × next(v) → stopped` if project throws.
- Error, complete, unsubscribe → `stopped`.

## 6. Output function (G : S × Z → A*)

- `active(i) × next(v) → next(project.call(thisArg, v, i))`.
- Project throw → `error(e)`.
- `error(e) → error(e)`, `complete → complete`.

## Worked trace

`of(10, 20).pipe(map((v, i) => v + i))` writes `next(10) next(21) complete`.

## Why this is Mealy rather than Moore

`G(active(i), next(v))` depends on both the index state and the value input. A Moore machine would need a state per pending output value.

## Edge cases fixed by the 7.x source

- Index counts source emissions, not downstream subscribers.
- Errors from the source do not call `project`.

## Source anchors

- `src/internal/operators/map.ts`: `let index = 0` and `project.call(thisArg, value, index++)` inside `createOperatorSubscriber`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
