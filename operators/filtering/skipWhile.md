# `skipWhile` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipWhile.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `skipWhile(predicate: (value, index) => boolean): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Drop values while `predicate` is true. The first value that fails the predicate, and everything after it, is emitted. The predicate is not consulted again after that.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { skipping(i), forwarding, stopped }`.

## 2. Initial state (S0)

`skipping(0)`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `skipping × next → skipping(i+1)` if predicate true.
- `skipping × next → forwarding` if predicate false.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- Skipped next → `ε`.
- The failing next and later nexts → `next(v)`.

## Worked trace

`skipWhile(x => x < 3)` on `1 2 3 1` writes `next(3) next(1) complete`. The later 1 passes.

## Why this is Mealy rather than Moore

Predicate on the input flips state and also decides whether that same input is in the output word.

## Edge cases fixed by the 7.x source

- Index increments only while skipping.
- Once forwarding, predicate throws cannot happen because it is not called.

## Source anchors

- `src/internal/operators/skipWhile.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
