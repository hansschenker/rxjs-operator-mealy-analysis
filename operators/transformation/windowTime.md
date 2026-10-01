# `windowTime` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowTime.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `windowTime(windowTimeSpan, windowCreationInterval?, maxWindowSize?, scheduler?): OperatorFunction<T, Observable<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Window analogue of `bufferTime`. Span timers close windows; optional creation interval opens extra windows; optional max size closes early.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { open windows with timers, stopped }`.

## 2. Initial state (S0)

First window open and emitted.

## 3. Input alphabet (Z)

`{ next(v), error, complete, spanTick, creationTick, unsubscribe }`.

## 4. Output alphabet (A)

Window observables and their notifications.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Ticks close or open windows.
- `next` forwards into open windows and may hit max size.

## 6. Output function (G : S × Z → A*)

- Open action → outer `next(window$)`.
- Source next → inner nexts.
- Span tick → inner complete, and maybe a new outer next.

## Worked trace

`windowTime(1000)` emits a new window observable every second and completes the previous one, even if it saw no values.

## Why this is Mealy rather than Moore

Timer inputs, not source values, decide window boundaries. Output word of a tick reads which window is open.

## Edge cases fixed by the 7.x source

- Empty windows are still emitted and completed.
- Scheduler owns the ticks.

## Source anchors

- `src/internal/operators/windowTime.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
