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
