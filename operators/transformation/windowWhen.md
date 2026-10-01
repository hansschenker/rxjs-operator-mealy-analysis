# `windowWhen` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowWhen.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `windowWhen(closingSelector: () => Observable<any>): OperatorFunction<T, Observable<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`windowWhen` is a pipeable operator on the RxJS 7.x line. Stable. Window analogue of `bufferWhen`. A window is open from subscribe. When the closing observable emits, complete the window, emit a new one, and call `closingSelector` again.

In plain terms, the operator keeps this memory: S = { open window, stopped }. At subscription, before any source notification, that memory is First window emitted, closer subscribed. It reacts to these events: { next(v), error, complete, closeNext, closeError, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two closer emissions produce three windows if the source is still active after the second close (the third is the newly opened one).

Details that a marble diagram often leaves out: `closingSelector` runs per window. Selector throw is an error.

## Role in the notification machine

Window analogue of `bufferWhen`. A window is open from subscribe. When the closing observable emits, complete the window, emit a new one, and call `closingSelector` again.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { open window, stopped }`.

## 2. Initial state (S0)

First window emitted, closer subscribed.

## 3. Input alphabet (Z)

`{ next(v), error, complete, closeNext, closeError, unsubscribe }`.

## 4. Output alphabet (A)

Window observables and inner notifications.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `closeNext` swaps the open window.
- Source next keeps it.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `subscribe → next(window$)`.
- `next(v) →` inner next.
- `closeNext →` inner complete · outer `next(newWindow$)`.
- `complete →` inner complete · outer complete.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { open window, stopped }.

Memory at subscribe: First window emitted, closer subscribed.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| closeNext swaps the open window | closeNext swaps the open window | next(window$) |
| Source next keeps it | Source next keeps it | inner next |
| Terminal | stopped | inner complete · outer next(newWindow$) |
| complete | named by the output row; memory change is in the transition rows above | inner complete · outer complete |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Two closer emissions produce three windows if the source is still active after the second close (the third is the newly opened one).

## Why this is Mealy rather than Moore

Close input writes a two-letter word (complete old, emit new) from the single open-window state.

## Edge cases fixed by the 7.x source

- `closingSelector` runs per window.
- Selector throw is an error.

## Source anchors

- `src/internal/operators/windowWhen.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
