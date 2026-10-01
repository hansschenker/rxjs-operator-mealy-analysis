# `bufferWhen` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferWhen.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `bufferWhen(closingSelector: () => Observable<any>): OperatorFunction<T, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`bufferWhen` is a pipeable operator on the RxJS 7.x line. Stable. One buffer is open. Subscribe to `closingSelector()` immediately. When that notifier emits, emit the buffer, open a new one, and subscribe to a fresh closer. Notifier complete also closes. Source complete emits the open buffer and completes.

In plain terms, the operator keeps this memory: S = { open(buf), stopped }. At subscription, before any source notification, that memory is open([]) with the first closer subscribed. It reacts to these events: { next(v), error, complete, closeNext, closeError, closeComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A closer that emits twice produces two arrays covering the values between those signals, then arms a third buffer.

Details that a marble diagram often leaves out: `closingSelector` is invoked per buffer, not once. A throw from the selector is an error output.

## Role in the notification machine

One buffer is open. Subscribe to `closingSelector()` immediately. When that notifier emits, emit the buffer, open a new one, and subscribe to a fresh closer. Notifier complete also closes. Source complete emits the open buffer and completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { open(buf), stopped }`.

## 2. Initial state (S0)

`open([])` with the first closer subscribed.

## 3. Input alphabet (Z)

`{ next(v), error, complete, closeNext, closeError, closeComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T[]), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v) → open(buf ++ [v])`.
- `closeNext` or `closeComplete → open([])` with a new closer, unless the source has already ended.
- Source complete or any error → `stopped`.

## 6. Output function (G : S × Z → A*)

- `closeNext` / `closeComplete → next(buf)`.
- `complete → next(buf) · complete`.
- `error → error`.
- `next → ε`.

## Worked trace

A closer that emits twice produces two arrays covering the values between those signals, then arms a third buffer.

## Why this is Mealy rather than Moore

Close is an input that reads the buffer state into the output word and resets it.

## Edge cases fixed by the 7.x source

- `closingSelector` is invoked per buffer, not once.
- A throw from the selector is an error output.

## Source anchors

- `src/internal/operators/bufferWhen.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
