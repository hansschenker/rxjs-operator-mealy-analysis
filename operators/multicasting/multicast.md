# `multicast` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/multicast.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `multicast(subjectOrFactory, selector?): OperatorFunction<T, R>` |
| Status on the 7.x line | Deprecated on 7.x in favor of `share` / `connect`. Still in the operator directory. |

SuperGrok is the main contributor of this analysis.

## Explanation

`multicast` is a pipeable connectable operator on the RxJS 7.x line. Deprecated on 7.x in favor of `share` / `connect`. Still in the operator directory. Share one subscription to the source through a Subject. Without a selector, the result is a `ConnectableObservable`: subscribers attach to the subject, and `connect()` subscribes the subject to the source. With a selector, the subject is wired for the duration of the selector's observable.

In plain terms, the operator keeps this memory: S = { disconnected, connected, stopped }. Subject memory (replay or not) is a parameter of the subject, modeled as extra state if the subject is a ReplaySubject or BehaviorSubject. At subscription, before any source notification, that memory is disconnected. It reacts to these events: { subscriberJoin, subscriberLeave, connect, sourceNext, sourceError, sourceComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two subscribers and one connect() cause one source subscription. Both receive later nexts. Without connect, neither receives source values.

Details that a marble diagram often leaves out: Factory form builds a fresh subject per connectable subscription. Passing a subject instance shares that subject. Deprecated; `share` covers refcounted use.

## Role in the notification machine

Share one subscription to the source through a Subject. Without a selector, the result is a `ConnectableObservable`: subscribers attach to the subject, and `connect()` subscribes the subject to the source. With a selector, the subject is wired for the duration of the selector's observable.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { disconnected, connected, stopped }`. Subject memory (replay or not) is a parameter of the subject, modeled as extra state if the subject is a ReplaySubject or BehaviorSubject.

## 2. Initial state (S0)

`disconnected`.

## 3. Input alphabet (Z)

`{ subscriberJoin, subscriberLeave, connect, sourceNext, sourceError, sourceComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }` toward subject subscribers.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `connect` in disconnected → `connected` and subscribe source to subject.
- `sourceNext` stays connected.
- Source terminal → `stopped` (subject terminated).
- Selector form connects while the selected observable is subscribed.

## 6. Output function (G : S × Z → A*)

- `sourceNext →` subject `next` (fan-out is the subject's job).
- Before connect, source symbols are not produced.
- Late subscribers see whatever the subject replays.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { disconnected, connected, stopped }. Subject memory (replay or not) is a parameter of the subject, modeled as extra state if the subject is a ReplaySubject or BehaviorSubject.

Memory at subscribe: disconnected.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| connect in disconnected | connected and subscribe source to subject | subject next (fan-out is the subject's job) |
| sourceNext stays connected | sourceNext stays connected | nothing named on a separate output row |
| Source terminal | stopped (subject terminated) | Before connect, source symbols are not produced |
| Selector form connects while the selected observable is subscribed | Selector form connects while the selected observable is subscribed | Late subscribers see whatever the subject replays |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Two subscribers and one `connect()` cause one source subscription. Both receive later nexts. Without connect, neither receives source values.

## Why this is Mealy rather than Moore

`connect` is an input that arms forwarding. Source next writes to subscribers only in `connected`.

## Edge cases fixed by the 7.x source

- Factory form builds a fresh subject per connectable subscription.
- Passing a subject instance shares that subject.
- Deprecated; `share` covers refcounted use.

## Source anchors

- `src/internal/operators/multicast.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
