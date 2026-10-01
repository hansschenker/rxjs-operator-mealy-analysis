# `last` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/last.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `last(predicate?, defaultValue?): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Remember the latest matching value. On source complete, emit it and complete. If none matched, emit default or `EmptyError`. Must see complete; it cannot emit early.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { seen(v) | empty, stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Matching next → `seen(v)`.
- Non-matching next stays.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete` in `seen(v) → next(v) · complete`.
- `complete` in `empty → next(default) · complete` or `error(EmptyError)`.

## Worked trace

`of(1, 2, 3).pipe(last())` writes `next(3) complete` only after the source completes.

## Why this is Mealy rather than Moore

Complete input writes the remembered value. Next input writes `ε` and updates memory.

## Edge cases fixed by the 7.x source

- Does not unsubscribe early.
- Predicate optional.

## Source anchors

- `src/internal/operators/last.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
