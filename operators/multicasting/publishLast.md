# `publishLast` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publishLast.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `publishLast(): OperatorFunction<T, T>` |
| Status on the 7.x line | Deprecated. AsyncSubject-backed multicast. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subject is an `AsyncSubject`. It emits only the last source value, and only when the source completes, to current and late subscribers. Error is sticky.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { disconnected, connected(last | ⊥), stopped(last | error) }`.

## 2. Initial state (S0)

`disconnected`, no last.

## 3. Input alphabet (Z)

`multicast` alphabet.

## 4. Output alphabet (A)

`{ next(last), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source next overwrites last.
- Source complete → stopped and subject emits.
- Late join after stop still reads the AsyncSubject cache.

## 6. Output function (G : S × Z → A*)

- Source next → `ε` to subscribers.
- Source complete → `next(last) · complete` if a last exists, else `complete`.
- Late `subscriberJoin` after stop replays that word.

## Worked trace

Source `1, 2, complete` after connect writes `next(2) complete` to subscribers. The `1` is overwritten.

## Why this is Mealy rather than Moore

Complete input writes the stored last. Next input writes `ε` and updates last. AsyncSubject replay is `G(stopped, subscriberJoin)`.

## Edge cases fixed by the 7.x source

- No value before complete.
- Deprecated.

## Source anchors

- `src/internal/operators/publishLast.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
