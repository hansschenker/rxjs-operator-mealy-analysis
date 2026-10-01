# Test plan: `webSocket`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/webSocket.md](../../operators/creation/webSocket.md).

| | |
|---|---|
| Category | Creation |
| Kind | Subject-like socket factory |
| RxJS 7.x source | `src/internal/observable/dom/webSocket.ts` |
| Signature | `webSocket(urlOrConfig): WebSocketSubject<T>` |
| Status | Stable. |

## Behavior under test

Returns a WebSocketSubject. The first subscriber opens the socket. Incoming messages are nexts to all subscribers. `next` on the subject sends a frame. Complete closes the socket. Error from the socket errors subscribers. The last unsubscribe closes it. Config can multiplex by a subMsg/unsubMsg protocol.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ closed, opening, open(refCount), stopped }`. Outgoing queue exists while opening.

Memory at subscribe: `closed`.

Events: `{ subscriberJoin, subscriberLeave, send(v), socketMessage, socketError, socketClose, complete, unsubscribe }`.

Possible notifications: `{ next(message), error, complete }` plus socket send actions.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `First join → opening, then open.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Further joins increment refCount.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Last leave → closed and socket close.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `socketClose or socketError → stopped.`
6. Transition 5: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `send while opening queues; while open writes to the socket.`
7. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `socketMessage → next to subscribers.`
8. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `send → ε downstream and a socket send action.`
9. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `socketError → error.`
10. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `complete on the subject → complete to subscribers and close the socket.`
11. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
12. Source edge 1: `Deserializer and serializer are config parameters of G.`
13. Source edge 2: `Multiplex uses subMsg on join and unsubMsg on leave.`
14. Source edge 3: `Not under `src/internal/operators`; it is a DOM creation subject.`

## Sequence from the analysis

Two subscribers share one socket. A message writes one next to each. The last unsubscribe closes the socket.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
