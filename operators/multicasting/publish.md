# `publish` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publish.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `publish(selector?): OperatorFunction<T, T>` |
| Status on the 7.x line | Deprecated. `publish()` is `multicast(() => new Subject())`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`publish` is a pipeable connectable operator on the RxJS 7.x line. Deprecated. `publish()` is `multicast(() => new Subject())`. ConnectableObservable backed by a plain Subject. No replay, no initial value. Subscribers see only nexts after they subscribe and after `connect()`.

In plain terms, the operator keeps this memory: S = { disconnected, connected, stopped }. Subject has no memory. At subscription, before any source notification, that memory is disconnected. It reacts to these events: Same as multicast.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Connect, source emits 1, late subscriber joins, source emits 2. Late subscriber sees only 2.

Details that a marble diagram often leaves out: Must call `connect()` or `refCount()`. Error on the subject is sticky for late subscribers.

## Role in the notification machine

ConnectableObservable backed by a plain Subject. No replay, no initial value. Subscribers see only nexts after they subscribe and after `connect()`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { disconnected, connected, stopped }`. Subject has no memory.

## 2. Initial state (S0)

`disconnected`.

## 3. Input alphabet (Z)

Same as `multicast`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Same connect/disconnect structure as `multicast` with a Subject factory.

## 6. Output function (G : S × Z → A*)

- Source next while connected → `next` to current subject subscribers only.
- No replay on subscriberJoin.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { disconnected, connected, stopped }. Subject has no memory.

Memory at subscribe: disconnected.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Same connect/disconnect structure as multicast with a Subject factory | Same connect/disconnect structure as multicast with a Subject factory | next to current subject subscribers only |
| No replay on subscriberJoin | named by the output row; memory change is in the transition rows above | No replay on subscriberJoin |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Connect, source emits 1, late subscriber joins, source emits 2. Late subscriber sees only 2.

## Why this is Mealy rather than Moore

Join input writes `ε` because a Subject has no buffer. That is a Mealy fact about `(disconnected-or-connected, subscriberJoin)`.

## Edge cases fixed by the 7.x source

- Must call `connect()` or `refCount()`.
- Error on the subject is sticky for late subscribers.

## Source anchors

- `src/internal/operators/publish.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
