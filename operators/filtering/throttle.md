# `throttle` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/throttle.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `throttle(durationSelector, config?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. Config `{ leading, trailing }` added on the 7.x line. |

SuperGrok is the main contributor of this analysis.

## Explanation

`throttle` is a pipeable operator on the RxJS 7.x line. Stable. Config `{ leading, trailing }` added on the 7.x line. Leading-edge by default: the first value emits immediately and starts `durationSelector(value)`. Values during the duration update a trailing buffer. When the duration ends, if trailing is enabled and a value was buffered, emit it and start a new duration. Default config is leading true, trailing false.

In plain terms, the operator keeps this memory: S = { idle, throttling(trailingValue | ⊥), stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { next(v), error, complete, durationNext, durationComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Default throttle: values 1, 2, 3 inside one duration write only next(1).

Details that a marble diagram often leaves out: `leading: false, trailing: true` becomes audit-like. Duration selector receives the value that opened the window.

## Role in the notification machine

Leading-edge by default: the first value emits immediately and starts `durationSelector(value)`. Values during the duration update a trailing buffer. When the duration ends, if trailing is enabled and a value was buffered, emit it and start a new duration. Default config is leading true, trailing false.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, throttling(trailingValue | ⊥), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, durationNext, durationComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × next → throttling` (leading emit already decided by G).
- `throttling × next` stores trailing value.
- Duration end → `idle`, or back to throttling if a trailing emit starts a new window.
- Complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- Leading next in `idle → next(v)`.
- Next while throttling → `ε`.
- Duration end → `next(trailing)` if config.trailing and a value is buffered, else `ε`.
- Complete may emit the trailing value when trailing is set.

## Worked trace

Default throttle: values 1, 2, 3 inside one duration write only `next(1)`.

## Why this is Mealy rather than Moore

Leading emit is `G(idle, next)`. Trailing emit is `G(throttling, durationEnd)`. Same value family, different input, different timing.

## Edge cases fixed by the 7.x source

- `leading: false, trailing: true` becomes audit-like.
- Duration selector receives the value that opened the window.

## Source anchors

- `src/internal/operators/throttle.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
