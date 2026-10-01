# `windowToggle` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/windowToggle.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `windowToggle(openings, closingSelector): OperatorFunction<T, Observable<T>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`windowToggle` is a pipeable operator on the RxJS 7.x line. Stable. Window analogue of `bufferToggle`. Each opening emits a new window observable and subscribes to a closer. Source values go to every open window. Closer next completes that window.

In plain terms, the operator keeps this memory: S = { list of open windows, stopped }. At subscription, before any source notification, that memory is No window until the first opening. It reacts to these events: { next(v), error, complete, opening, closing_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: One opening, two values, one closing: one window observable emits both values and completes.

Details that a marble diagram often leaves out: Values with no open window are dropped. Closing complete without a value does not emit a boundary by itself in the same way a next does; it unsubscribes the closer.

## Role in the notification machine

Window analogue of `bufferToggle`. Each opening emits a new window observable and subscribes to a closer. Source values go to every open window. Closer next completes that window.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { list of open windows, stopped }`.

## 2. Initial state (S0)

No window until the first opening.

## 3. Input alphabet (Z)

`{ next(v), error, complete, opening, closing_i, unsubscribe }`.

## 4. Output alphabet (A)

Outer window observables; inner notifications.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `opening` appends a window.
- `closing_i` removes it.
- `next` does not change membership.

## 6. Output function (G : S × Z → A*)

- `opening → next(window$)`.
- `next(v) →` inner next on each open window.
- `closing_i →` that window `complete`.
- Source complete completes open windows and the outer.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { list of open windows, stopped }.

Memory at subscribe: No window until the first opening.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| opening appends a window | opening appends a window | next(window$) |
| closing_i removes it | closing_i removes it | inner next on each open window |
| next does not change membership | next does not change membership | nothing named on a separate output row |
| closing_i | named by the output row; memory change is in the transition rows above | that window complete |
| Source complete completes open windows and the outer | named by the output row; memory change is in the transition rows above | Source complete completes open windows and the outer |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

One opening, two values, one closing: one window observable emits both values and completes.

## Why this is Mealy rather than Moore

Opening and closing inputs write outer or inner terminal letters; source next writes inner letters. Membership state selects the targets.

## Edge cases fixed by the 7.x source

- Values with no open window are dropped.
- Closing complete without a value does not emit a boundary by itself in the same way a next does; it unsubscribes the closer.

## Source anchors

- `src/internal/operators/windowToggle.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
