# `timeInterval` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timeInterval.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `timeInterval(scheduler = asyncScheduler): OperatorFunction<T, TimeInterval<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`timeInterval` is a pipeable operator on the RxJS 7.x line. Stable. Emit `{ value, interval }` where `interval` is scheduler time since the previous emission, or since subscription for the first value.

In plain terms, the operator keeps this memory: S = { active(lastTime), stopped }. At subscription, before any source notification, that memory is active(scheduler.now() at subscribe). It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two values one second apart write intervals near 0 (or subscribe delay) and near 1000.

Details that a marble diagram often leaves out: Uses the scheduler clock, not `Date.now` directly, when a scheduler is passed. Complete is not wrapped.

## Role in the notification machine

Emit `{ value, interval }` where `interval` is scheduler time since the previous emission, or since subscription for the first value.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active(lastTime), stopped }`.

## 2. Initial state (S0)

`active(scheduler.now()` at subscribe`)`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(TimeInterval), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next` replaces `lastTime` with now.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → next({ value: v, interval: now - lastTime })`.
- Error and complete copy through.

## Worked trace

Two values one second apart write intervals near 0 (or subscribe delay) and near 1000.

## Why this is Mealy rather than Moore

Interval in the output is computed from state `lastTime` and the arrival input's now.

## Edge cases fixed by the 7.x source

- Uses the scheduler clock, not `Date.now` directly, when a scheduler is passed.
- Complete is not wrapped.

## Source anchors

- `src/internal/operators/timeInterval.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
