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
