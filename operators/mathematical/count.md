# `count` — Mealy 6-tuple

| | |
|---|---|
| Category | Mathematical and Aggregate |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/count.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `count(predicate?): OperatorFunction<T, number>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Count source nexts, or count those matching `predicate`. Emit the count on complete, then complete. No emission before complete.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { counting(n, i), stopped }`.

## 2. Initial state (S0)

`counting(0, 0)`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(number), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Matching next increments n and i.
- Non-matching increments i only.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete → next(n) · complete`.
- Predicate throw → `error`.

## Worked trace

`of(1, 2, 3, 4).pipe(count(x => x % 2 === 0))` writes `next(2) complete`.

## Why this is Mealy rather than Moore

Complete input writes the counter state. Next input only updates it.

## Edge cases fixed by the 7.x source

- Empty source emits `next(0) complete`.
- Predicate receives index.

## Source anchors

- `src/internal/operators/count.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
