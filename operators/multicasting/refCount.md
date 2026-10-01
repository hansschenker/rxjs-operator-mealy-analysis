# `refCount` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Connectable adapter |
| RxJS 7.x source | `src/internal/operators/refCount.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `refCount(): OperatorFunction<T, T>` |
| Status on the 7.x line | Deprecated. Use `share`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`refCount` is a connectable adapter on the RxJS 7.x line. Deprecated. Use `share`. For a ConnectableObservable, the first subscriber calls `connect()`, further subscribers increment a count, and the last unsubscribe disconnects. Late subscribers do not replay unless the underlying subject does.

In plain terms, the operator keeps this memory: { refCount, connection | ⊥, stopped }. At subscription, before any source notification, that memory is refCount 0, no connection. It reacts to these events: { subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two subscribers share one connection. When both leave, the source is unsubscribed.

Details that a marble diagram often leaves out: Deprecated in favor of `share`. Disconnect does not reset a sticky subject error by itself.

## Role in the notification machine

For a ConnectableObservable, the first subscriber calls `connect()`, further subscribers increment a count, and the last unsubscribe disconnects. Late subscribers do not replay unless the underlying subject does.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ refCount, connection | ⊥, stopped }`.

## 2. Initial state (S0)

refCount 0, no connection.

## 3. Input alphabet (Z)

`{ subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Join at 0 → connect, refCount 1.
- Join increments.
- Leave decrements; at 0 disconnect.
- Source terminal stops the subject.

## 6. Output function (G : S × Z → A*)

- Source next → next to current subscribers.
- Join writes ε unless the subject replays.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { refCount, connection \| ⊥, stopped }.

Memory at subscribe: refCount 0, no connection.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Join at 0 | connect, refCount 1 | next to current subscribers |
| Join increments | Join increments | Join writes ε unless the subject replays |
| Leave decrements; at 0 disconnect | Leave decrements; at 0 disconnect | nothing named on a separate output row |
| Source terminal stops the subject | Source terminal stops the subject | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Two subscribers share one connection. When both leave, the source is unsubscribed.

## Why this is Mealy rather than Moore

Join and leave are inputs that change connection without themselves being value letters. Source next writes a letter only while connected.

## Edge cases fixed by the 7.x source

- Deprecated in favor of `share`.
- Disconnect does not reset a sticky subject error by itself.

## Source anchors

- `src/internal/operators/refCount.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
