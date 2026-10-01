# `flatMap` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Deprecated alias |
| RxJS 7.x source | `src/internal/operators/flatMap.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `flatMap(project, resultSelector?, concurrent?): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated alias of `mergeMap`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

`flatMap.ts` re-exports `mergeMap`. Concurrency defaults to Infinity. Inners are subscribed as outer values arrive, up to the limit, and their nexts are interleaved.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Same as `mergeMap`: active count, queue, outerDone, stopped.

## 2. Initial state (S0)

Active 0, empty queue.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Identical to `mergeMap`.

## 6. Output function (G : S × Z → A*)

- Identical to `mergeMap`.

## Worked trace

Same word as `mergeMap` on the same projection.

## Why this is Mealy rather than Moore

Alias only. The complete-or-ε decision on innerComplete still reads active count and outerDone.

## Edge cases fixed by the 7.x source

- File is a re-export.
- Prefer `mergeMap`.

## Source anchors

- `src/internal/operators/flatMap.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
