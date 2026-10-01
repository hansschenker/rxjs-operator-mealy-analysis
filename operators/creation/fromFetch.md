# `fromFetch` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold fetch producer |
| RxJS 7.x source | `src/internal/observable/dom/fetch.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `fromFetch(input, init?): Observable<Response>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`fromFetch` is a cold fetch producer on the RxJS 7.x line. Stable. Subscribe calls `fetch`. The Response is a single next, then complete. Unsubscribe aborts via AbortController. A fetch rejection is an error. The body is not read unless the caller reads the Response.

In plain terms, the operator keeps this memory: { idle, inflight(controller), stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, resolve(response), reject(error), unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A successful fetch writes next(response) complete. Unsubscribing before resolve aborts and writes nothing.

Details that a marble diagram often leaves out: Selector overload can project the Response to another observable; that projection is then flattened. Cold: one fetch per subscription.

## Role in the notification machine

Subscribe calls `fetch`. The Response is a single next, then complete. Unsubscribe aborts via AbortController. A fetch rejection is an error. The body is not read unless the caller reads the Response.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ idle, inflight(controller), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, resolve(response), reject(error), unsubscribe }`.

## 4. Output alphabet (A)

`{ next(Response), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Subscribe → inflight.
- Resolve or reject → stopped.
- Unsubscribe → stopped and abort.

## 6. Output function (G : S × Z → A*)

- Resolve → next(response) · complete.
- Reject → error.
- Unsubscribe → ε.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { idle, inflight(controller), stopped }.

Memory at subscribe: idle.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Subscribe | inflight | ε |
| Resolve or reject | stopped | next(response) · complete |
| Unsubscribe | stopped and abort | nothing named on a separate output row |
| Reject | named by the output row; memory change is in the transition rows above | error |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

A successful fetch writes `next(response) complete`. Unsubscribing before resolve aborts and writes nothing.

## Why this is Mealy rather than Moore

Resolve and reject are different inputs in `inflight` and write different words. Abort moves to stopped with ε.

## Edge cases fixed by the 7.x source

- Selector overload can project the Response to another observable; that projection is then flattened.
- Cold: one fetch per subscription.

## Source anchors

- `src/internal/observable/dom/fetch.ts` exports `fromFetch`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
