# `pairwise` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/pairwise.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `pairwise(): OperatorFunction<T, [T, T]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Remember the previous value. The first next only stores. From the second next on, emit `[previous, current]` and shift memory. Complete and error pass through. A single-value source completes with no emission.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { empty, holding(prev), stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next([T, T]), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `empty × next(v) → holding(v)`.
- `holding(p) × next(v) → holding(v)`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `empty × next → ε`.
- `holding(p) × next(v) → next([p, v])`.
- `complete → complete`, `error → error`.

## Worked trace

`of(1, 2, 3).pipe(pairwise())` writes `next([1, 2]) next([2, 3]) complete`.

## Why this is Mealy rather than Moore

The same `next` symbol writes `ε` in `empty` and a pair in `holding`. Textbook Mealy.

## Edge cases fixed by the 7.x source

- No emission on complete for a dangling first value.
- Pairs overlap by one.

## Source anchors

- `src/internal/operators/pairwise.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
