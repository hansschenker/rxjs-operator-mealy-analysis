# `timeout` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timeout.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `timeout(configOrDue, scheduler?): OperatorFunction<T, T | R>` |
| Status on the 7.x line | Stable. 7.x config form `{ each, first, with, meta, scheduler }` is the full machine. Numeric due is shorthand for `each`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`timeout` is a pipeable operator on the RxJS 7.x line. Stable. 7.x config form `{ each, first, with, meta, scheduler }` is the full machine. Numeric due is shorthand for `each`. Arm a timer on subscribe (`first`) and after each next (`each`). If the timer fires before the next source signal, either error with `TimeoutError` or switch to the `with` observable. A source next resets the each-timer.

In plain terms, the operator keeps this memory: S = { waiting(timer), forwardingReplacement, stopped }. At subscription, before any source notification, that memory is waiting with the first-timer armed. It reacts to these events: { next, error, complete, timeoutTick, replacementNext, replacementError, replacementComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: each: 1000 with a silent source writes error(TimeoutError) about a second after subscribe if first is also exceeded.

Details that a marble diagram often leaves out: `with` changes the timeout from a terminal error into a switch. `meta` is attached to TimeoutError. Absolute dates are allowed for first.

## Role in the notification machine

Arm a timer on subscribe (`first`) and after each next (`each`). If the timer fires before the next source signal, either error with `TimeoutError` or switch to the `with` observable. A source next resets the each-timer.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { waiting(timer), forwardingReplacement, stopped }`.

## 2. Initial state (S0)

`waiting` with the first-timer armed.

## 3. Input alphabet (Z)

`{ next, error, complete, timeoutTick, replacementNext, replacementError, replacementComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error(TimeoutError), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next` cancels and rearms the each-timer.
- `timeoutTick → stopped` if no `with`, else `forwardingReplacement`.
- Source complete → `stopped` and cancels the timer.

## 6. Output function (G : S × Z → A*)

- `next → next` and rearm action.
- `timeoutTick → error(TimeoutError)` or `ε` plus switch.
- Replacement notifications copy through.

## Worked trace

`each: 1000` with a silent source writes `error(TimeoutError)` about a second after subscribe if `first` is also exceeded.

## Why this is Mealy rather than Moore

Timeout tick writes an error or a switch depending on config and waiting state. Source next writes the value and refreshes the timer.

## Edge cases fixed by the 7.x source

- `with` changes the timeout from a terminal error into a switch.
- `meta` is attached to TimeoutError.
- Absolute dates are allowed for first.

## Source anchors

- `src/internal/operators/timeout.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
