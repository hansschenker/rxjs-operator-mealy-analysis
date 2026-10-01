# `take` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/take.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `take(count: number): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Emit the first `count` source nexts and complete, unsubscribing from the source. `take(0)` completes immediately and does not subscribe to the source. A shorter source just completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { counting(k), stopped }` for `0 ≤ k ≤ count`.

## 2. Initial state (S0)

`counting(0)`.

## 3. Input alphabet (Z)

`{ subscribe, next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `counting(k) × next → counting(k+1)` if `k+1 < count`, else `stopped`.
- `subscribe` with count 0 → `stopped` without a source subscription.
- Source complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v)` when `k+1 < count → next(v)`.
- `next(v)` when `k+1 = count → next(v) · complete`.
- `take(0)` subscribe → `complete`.
- Early source complete → `complete`.

## Worked trace

`take(2)` on `a b c` writes `next(a) next(b) complete` and cancels before `c`.

## Why this is Mealy rather than Moore

The count-reaching next writes two letters, `next·complete`. Earlier nexts write one. The input is the same kind; state picks the word length.

## Edge cases fixed by the 7.x source

- `count < 0` is treated as zero in 7.x and completes immediately.
- Unsubscribes on the completing next.

## Source anchors

- `src/internal/operators/take.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
