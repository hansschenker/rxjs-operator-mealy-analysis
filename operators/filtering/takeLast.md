# `takeLast` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/takeLast.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `takeLast(count: number): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Buffer at most `count` values. On source complete, emit the buffer in order and complete. Error passes through without emitting the buffer.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { buffer (ring of size ≤ count), stopped }`.

## 2. Initial state (S0)

Empty buffer.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next` pushes, dropping the oldest if over count.
- `complete → stopped`.
- `error → stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete → next(b1)…next(bn) · complete`.
- `error → error` with no flush.

## Worked trace

`takeLast(2)` on `1 2 3 4` writes `next(3) next(4) complete` after the source completes.

## Why this is Mealy rather than Moore

Complete input expands the buffer state into a word. Next input writes `ε`.

## Edge cases fixed by the 7.x source

- `count <= 0` completes with no values once the source completes.
- Must wait for complete, so it does not work on a non-completing source.

## Source anchors

- `src/internal/operators/takeLast.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
