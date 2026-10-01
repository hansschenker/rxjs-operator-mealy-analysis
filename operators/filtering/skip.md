# `skip` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skip.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `skip(count: number): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Drop the first `count` source nexts, then mirror the source. Error and complete always pass through.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { skipping(k), forwarding, stopped }` for `0 ≤ k ≤ count`.

## 2. Initial state (S0)

`skipping(0)` if count > 0, else `forwarding`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `skipping(k) × next → skipping(k+1)` while `k+1 < count`, else `forwarding`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `skipping × next → ε`.
- `forwarding × next → next(v)`.
- The next that reaches count is the first forwarded, or is still skipped depending on the off-by-one: after `count` skips, subsequent values emit. The value that increments k to count is skipped.

## Worked trace

`skip(2)` on `a b c d` writes `next(c) next(d) complete`.

## Why this is Mealy rather than Moore

The same next symbol is `ε` while skipping and `next(v)` while forwarding. State selects the word.

## Edge cases fixed by the 7.x source

- `skip(0)` is identity.
- Does not unsubscribe; it only drops.

## Source anchors

- `src/internal/operators/skip.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
