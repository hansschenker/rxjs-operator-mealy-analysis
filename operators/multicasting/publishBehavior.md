# `publishBehavior` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publishBehavior.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `publishBehavior(value: T): OperatorFunction<T, T>` |
| Status on the 7.x line | Deprecated. BehaviorSubject-backed multicast. |

SuperGrok is the main contributor of this analysis.

## Explanation

`publishBehavior` is a pipeable connectable operator on the RxJS 7.x line. Deprecated. BehaviorSubject-backed multicast. Like `publish`, but the subject is a `BehaviorSubject(value)`. Every new subscriber synchronously receives the current value, which starts as `value` even before connect.

In plain terms, the operator keeps this memory: S = { disconnected(current), connected(current), stopped }. At subscription, before any source notification, that memory is disconnected(initialValue). It reacts to these events: multicast alphabet plus the behavior current value.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: publishBehavior(0) subscriber before connect still gets next(0). After connect and source 1, subscribers get next(1).

Details that a marble diagram often leaves out: Initial value is emitted even if the source never emits. Deprecated.

## Role in the notification machine

Like `publish`, but the subject is a `BehaviorSubject(value)`. Every new subscriber synchronously receives the current value, which starts as `value` even before connect.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { disconnected(current), connected(current), stopped }`.

## 2. Initial state (S0)

`disconnected(initialValue)`.

## 3. Input alphabet (Z)

`multicast` alphabet plus the behavior current value.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `subscriberJoin` does not change connection, but current is readable.
- `sourceNext(v)` sets current to v if connected.

## 6. Output function (G : S × Z → A*)

- `subscriberJoin → next(current)` synchronously.
- `sourceNext(v) → next(v)` to all current subscribers.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { disconnected(current), connected(current), stopped }.

Memory at subscribe: disconnected(initialValue).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| subscriberJoin does not change connection, but current is readable | subscriberJoin does not change connection, but current is readable | next(current) synchronously |
| sourceNext(v) | sourceNext(v) sets current to v if connected | next(v) to all current subscribers |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`publishBehavior(0)` subscriber before connect still gets `next(0)`. After connect and source `1`, subscribers get `next(1)`.

## Why this is Mealy rather than Moore

Join input writes the state value. Source next writes the input value and updates state. Both are Mealy.

## Edge cases fixed by the 7.x source

- Initial value is emitted even if the source never emits.
- Deprecated.

## Source anchors

- `src/internal/operators/publishBehavior.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
