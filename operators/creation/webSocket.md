# `webSocket` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Subject-like socket factory |
| RxJS 7.x source | `src/internal/observable/dom/webSocket.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `webSocket(urlOrConfig): WebSocketSubject<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`webSocket` is a subject-like socket factory on the RxJS 7.x line. Stable. Returns a WebSocketSubject. The first subscriber opens the socket. Incoming messages are nexts to all subscribers. `next` on the subject sends a frame. Complete closes the socket. Error from the socket errors subscribers. The last unsubscribe closes it. Config can multiplex by a subMsg/unsubMsg protocol.

In plain terms, the operator keeps this memory: { closed, opening, open(refCount), stopped }. Outgoing queue exists while opening. At subscription, before any source notification, that memory is closed. It reacts to these events: { subscriberJoin, subscriberLeave, send(v), socketMessage, socketError, socketClose, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two subscribers share one socket. A message writes one next to each. The last unsubscribe closes the socket.

Details that a marble diagram often leaves out: Deserializer and serializer are config parameters of G. Multiplex uses subMsg on join and unsubMsg on leave. Not under `src/internal/operators`; it is a DOM creation subject.

## Role in the notification machine

Returns a WebSocketSubject. The first subscriber opens the socket. Incoming messages are nexts to all subscribers. `next` on the subject sends a frame. Complete closes the socket. Error from the socket errors subscribers. The last unsubscribe closes it. Config can multiplex by a subMsg/unsubMsg protocol.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ closed, opening, open(refCount), stopped }`. Outgoing queue exists while opening.

## 2. Initial state (S0)

`closed`.

## 3. Input alphabet (Z)

`{ subscriberJoin, subscriberLeave, send(v), socketMessage, socketError, socketClose, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(message), error, complete }` plus socket send actions.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- First join → opening, then open.
- Further joins increment refCount.
- Last leave → closed and socket close.
- socketClose or socketError → stopped.
- send while opening queues; while open writes to the socket.

## 6. Output function (G : S × Z → A*)

- socketMessage → next to subscribers.
- send → ε downstream and a socket send action.
- socketError → error.
- complete on the subject → complete to subscribers and close the socket.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { closed, opening, open(refCount), stopped }. Outgoing queue exists while opening.

Memory at subscribe: closed.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| First join | opening, then open | next to subscribers |
| Further joins increment refCount | Further joins increment refCount | ε downstream and a socket send action |
| Last leave | closed and socket close | error |
| socketClose or socketError | stopped | nothing named on a separate output row |
| send while opening queues; while open writes to the socket | send while opening queues; while open writes to the socket | complete to subscribers and close the socket |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Two subscribers share one socket. A message writes one next to each. The last unsubscribe closes the socket.

## Why this is Mealy rather than Moore

A socket message input writes next. A subject next input writes ε downstream and a send action. Same open state, different input, different word.

## Edge cases fixed by the 7.x source

- Deserializer and serializer are config parameters of G.
- Multiplex uses subMsg on join and unsubMsg on leave.
- Not under `src/internal/operators`; it is a DOM creation subject.

## Source anchors

- `src/internal/observable/dom/webSocket.ts` and `WebSocketSubject.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
