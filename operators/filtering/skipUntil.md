# `skipUntil` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/skipUntil.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `skipUntil(notifier: Observable<any>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`skipUntil` is a pipeable operator on the RxJS 7.x line. Stable. Drop source values until `notifier` emits once. Then unsubscribe the notifier and mirror the source. Notifier error is an error. Notifier complete without a next leaves the machine skipping forever until the source ends.

In plain terms, the operator keeps this memory: S = { skipping, forwarding, stopped }. At subscription, before any source notification, that memory is skipping. It reacts to these events: { next(v), error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source values before a click are dropped; the click itself is not emitted; later source values pass.

Details that a marble diagram often leaves out: Notifier value is ignored, only its arrival matters. Source complete while still skipping writes `complete` with no values.

## Role in the notification machine

Drop source values until `notifier` emits once. Then unsubscribe the notifier and mirror the source. Notifier error is an error. Notifier complete without a next leaves the machine skipping forever until the source ends.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { skipping, forwarding, stopped }`.

## 2. Initial state (S0)

`skipping`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `skipping × notifierNext → forwarding`.
- `skipping × next → skipping`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `skipping × next → ε`.
- `forwarding × next → next(v)`.
- `notifierNext → ε` (it only flips state).

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { skipping, forwarding, stopped }.

Memory at subscribe: skipping.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| skipping × notifierNext | forwarding | ε |
| skipping × next | skipping | next(v) |
| Terminal | stopped | ε (it only flips state) |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Source values before a click are dropped; the click itself is not emitted; later source values pass.

## Why this is Mealy rather than Moore

Source next writes `ε` or `next(v)` according to whether notifier input has already moved the state.

## Edge cases fixed by the 7.x source

- Notifier value is ignored, only its arrival matters.
- Source complete while still skipping writes `complete` with no values.

## Source anchors

- `src/internal/operators/skipUntil.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
