# `catchError` — Mealy 6-tuple

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/catchError.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `catchError(selector: (err, caught) => ObservableInput<T>): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`catchError` is a pipeable operator on the RxJS 7.x line. Stable. Forward source nexts and complete. On source error, call `selector(error, caught)` and subscribe to the returned observable instead. If the selector returns `caught` (the source observable passed in), this resubscribes. Selector throw is an error. After the switch, the replacement's notifications are forwarded.

In plain terms, the operator keeps this memory: S = { forwarding(source), forwarding(replacement), stopped }. At subscription, before any source notification, that memory is forwarding(source). It reacts to these events: { next, error, complete, replacementNext, replacementError, replacementComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source errors, selector returns of(0), output writes the source nexts so far, then next(0) complete, and no error.

Details that a marble diagram often leaves out: Returning `caught` is the retry-by-resubscribe pattern and can loop. Only errors are caught; complete is not a catch input.

## Role in the notification machine

Forward source nexts and complete. On source error, call `selector(error, caught)` and subscribe to the returned observable instead. If the selector returns `caught` (the source observable passed in), this resubscribes. Selector throw is an error. After the switch, the replacement's notifications are forwarded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { forwarding(source), forwarding(replacement), stopped }`.

## 2. Initial state (S0)

`forwarding(source)`.

## 3. Input alphabet (Z)

`{ next, error, complete, replacementNext, replacementError, replacementComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source next stays.
- Source error → `forwarding(replacement)` if selector returns, else `stopped`.
- Replacement terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- Source next → `next`.
- Source error → `ε` (switch action) or `error` if selector throws.
- Replacement notifications copy through.
- Source error is not forwarded as error when the selector handles it.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { forwarding(source), forwarding(replacement), stopped }.

Memory at subscribe: forwarding(source).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Source next stays | Source next stays | next |
| Source error | forwarding(replacement) if selector returns, else stopped | ε (switch action) or error if selector throws |
| Replacement terminal | stopped | Replacement notifications copy through |
| Source error is not forwarded as error when the selector handles it | named by the output row; memory change is in the transition rows above | Source error is not forwarded as error when the selector handles it |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Source errors, selector returns `of(0)`, output writes the source nexts so far, then `next(0) complete`, and no error.

## Why this is Mealy rather than Moore

Error input writes either a switch (`ε` plus a new subscription) or an error, depending on the selector result which is part of handling that input.

## Edge cases fixed by the 7.x source

- Returning `caught` is the retry-by-resubscribe pattern and can loop.
- Only errors are caught; complete is not a catch input.

## Source anchors

- `src/internal/operators/catchError.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
