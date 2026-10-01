# `timeoutWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/timeoutWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `timeoutWith(due, withObservable, scheduler?): OperatorFunction<T, T | R>` |
| Status on the 7.x line | Deprecated. Use `timeout({ each, with })`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`timeoutWith` is a pipeable operator on the RxJS 7.x line. Deprecated. Use `timeout({ each, with })`. Numeric timeout that switches to `withObservable` instead of erroring. Same waiting-timer machine as `timeout` with a replacement branch.

In plain terms, the operator keeps this memory: S = { waiting, switched, stopped }. At subscription, before any source notification, that memory is waiting. It reacts to these events: { next, error, complete, timeoutTick, innerNext, innerError, innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A silent source and timeoutWith(1000, of('fallback')) writes next('fallback') complete.

Details that a marble diagram often leaves out: Deprecated. Scheduler defaults to async.

## Role in the notification machine

Numeric timeout that switches to `withObservable` instead of erroring. Same waiting-timer machine as `timeout` with a replacement branch.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { waiting, switched, stopped }`.

## 2. Initial state (S0)

`waiting`.

## 3. Input alphabet (Z)

`{ next, error, complete, timeoutTick, innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Timeout tick → `switched` and subscribe to `with`.
- Source next rearms while waiting.

## 6. Output function (G : S × Z → A*)

- Timeout tick → `ε` (switch action).
- Inner notifications then copy through.
- No TimeoutError on the default path.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { waiting, switched, stopped }.

Memory at subscribe: waiting.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Timeout tick | switched and subscribe to with | ε (switch action) |
| Source next rearms while waiting | Source next rearms while waiting | nothing named on a separate output row |
| Inner notifications then copy through | named by the output row; memory change is in the transition rows above | Inner notifications then copy through |
| No TimeoutError on the default path | named by the output row; memory change is in the transition rows above | No TimeoutError on the default path |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

A silent source and `timeoutWith(1000, of('fallback'))` writes `next('fallback') complete`.

## Why this is Mealy rather than Moore

Timeout input selects the switch word. Deprecated alias of the `with` branch of `timeout`.

## Edge cases fixed by the 7.x source

- Deprecated.
- Scheduler defaults to async.

## Source anchors

- `src/internal/operators/timeoutWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
