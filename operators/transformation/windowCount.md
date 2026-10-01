# `windowCount` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowCount.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `windowCount(windowSize: number, startWindowEvery: number = windowSize): OperatorFunction<T, Observable<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Window analogue of `bufferCount`. Open windows of `windowSize` source values, starting a new window every `startWindowEvery` values. Emit each window observable when it opens. Complete a window when it reaches `windowSize`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { list of open windows with their counts, source count, stopped }`.

## 2. Initial state (S0)

First window emitted, count 0.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

Outer window observables; inner next/complete.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` increments counts, may complete full windows, may open a new window.
- Source complete completes open windows and stops.

## 6. Output function (G : S × Z → A*)

- On open: outer `next(window$)`.
- On source next: `next(v)` into each open window.
- On full: that window `complete`.

## Worked trace

`windowCount(2)` on `a b c d` emits two windows: `[a b]` and `[c d]`, each completed.

## Why this is Mealy rather than Moore

A source next may write only inner nexts, or inner nexts plus a window complete plus a new outer next, depending on the count state.

## Edge cases fixed by the 7.x source

- `startWindowEvery < windowSize` overlaps.
- `windowSize < 1` errors.

## Source anchors

- `src/internal/operators/windowCount.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
