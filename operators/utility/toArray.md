# `toArray` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/toArray.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `toArray(): OperatorFunction<T, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Buffer every source next. On complete, emit one array and complete. Error passes through and drops the buffer. Does not emit on an infinite source.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { collecting(buf), stopped }`.

## 2. Initial state (S0)

`collecting([])`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T[]), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` appends.
- `complete → stopped`.
- `error → stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete → next(buf) · complete`.
- `error → error`.

## Worked trace

`of(1, 2, 3).pipe(toArray())` writes `next([1, 2, 3]) complete`.

## Why this is Mealy rather than Moore

Complete input expands buffer state into a single next. Value inputs write `ε`.

## Edge cases fixed by the 7.x source

- Empty source emits `next([]) complete`.
- Unbounded memory.

## Source anchors

- `src/internal/operators/toArray.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
