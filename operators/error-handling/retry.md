# `retry` — Mealy 6-tuple

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/retry.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `retry(countOrConfig?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. 7.x accepts a number or `{ count, delay, resetOnSuccess }`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`retry` is a pipeable operator on the RxJS 7.x line. Stable. 7.x accepts a number or `{ count, delay, resetOnSuccess }`. On source error, resubscribe if retries remain. `count` is the number of resubscriptions (Infinity by default in the config object path; the numeric shorthand is the retry count). `delay` can be a duration or a notifier of the error. `resetOnSuccess` clears the attempt counter after a successful next. Complete passes through and does not retry.

In plain terms, the operator keeps this memory: S = { forwarding(attempt), waitingDelay, stopped }. At subscription, before any source notification, that memory is forwarding(0). It reacts to these events: { next, error, complete, delayTick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A source that errors twice with retry(1) writes the first attempt's nexts, suppresses the first error, resubscribes, then forwards the second error.

Details that a marble diagram often leaves out: Count Infinity retries forever. Delay notifier error becomes the output error and stops retries. `resetOnSuccess` changes the counter transition on next.

## Role in the notification machine

On source error, resubscribe if retries remain. `count` is the number of resubscriptions (Infinity by default in the config object path; the numeric shorthand is the retry count). `delay` can be a duration or a notifier of the error. `resetOnSuccess` clears the attempt counter after a successful next. Complete passes through and does not retry.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { forwarding(attempt), waitingDelay, stopped }`.

## 2. Initial state (S0)

`forwarding(0)`.

## 3. Input alphabet (Z)

`{ next, error, complete, delayTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next` stays forwarding; may reset attempt to 0.
- `error` → `waitingDelay` if attempts remain, else `stopped`.
- `delayTick` → `forwarding(attempt+1)` via resubscribe.
- `complete → stopped`.

## 6. Output function (G : S × Z → A*)

- `next → next`.
- `error → ε` if a retry will happen, else `error(e)`.
- `complete → complete`.
- Resubscribe is an action, not a notification.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { forwarding(attempt), waitingDelay, stopped }.

Memory at subscribe: forwarding(0).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next stays forwarding; may reset attempt to 0 | next stays forwarding; may reset attempt to 0 | next |
| error | waitingDelay if attempts remain, else stopped | ε if a retry will happen, else error(e) |
| delayTick | forwarding(attempt+1) via resubscribe | nothing named on a separate output row |
| complete | stopped | complete |
| Resubscribe is an action, not a notification | named by the output row; memory change is in the transition rows above | Resubscribe is an action, not a notification |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

A source that errors twice with `retry(1)` writes the first attempt's nexts, suppresses the first error, resubscribes, then forwards the second error.

## Why this is Mealy rather than Moore

Error input writes `ε` or `error(e)` depending on the attempt counter in state.

## Edge cases fixed by the 7.x source

- Count Infinity retries forever.
- Delay notifier error becomes the output error and stops retries.
- `resetOnSuccess` changes the counter transition on next.

## Source anchors

- `src/internal/operators/retry.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
