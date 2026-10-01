# `publishReplay` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publishReplay.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `publishReplay(bufferSize?, windowTime?, selector?, scheduler?): OperatorFunction<T, T>` |
| Status on the 7.x line | Deprecated. ReplaySubject-backed multicast. |

SuperGrok is the main contributor of this analysis.

## Explanation

`publishReplay` is a pipeable connectable operator on the RxJS 7.x line. Deprecated. ReplaySubject-backed multicast. ConnectableObservable over a `ReplaySubject(bufferSize, windowTime)`. Subscribers receive the buffered window on join, then live values after connect.

In plain terms, the operator keeps this memory: S = { disconnected(buffer), connected(buffer), stopped }. Buffer is a bounded queue. At subscription, before any source notification, that memory is disconnected(empty buffer). It reacts to these events: multicast alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Connect, emit 1 then 2, late subscriber joins: late subscriber gets next(1) next(2) from the buffer if bufferSize allows.

Details that a marble diagram often leaves out: Does not refCount unless composed with `refCount`. Deprecated in favor of `share({ connector: () => new ReplaySubject(...) })`.

## Role in the notification machine

ConnectableObservable over a `ReplaySubject(bufferSize, windowTime)`. Subscribers receive the buffered window on join, then live values after connect.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { disconnected(buffer), connected(buffer), stopped }`. Buffer is a bounded queue.

## 2. Initial state (S0)

`disconnected(empty buffer)`.

## 3. Input alphabet (Z)

`multicast` alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source next appends to the replay buffer while connected.
- Join does not change connection.

## 6. Output function (G : S × Z → A*)

- `subscriberJoin → next` for each buffered value still inside windowTime.
- `sourceNext → next` to live subscribers and a buffer append.

## Worked trace

Connect, emit 1 then 2, late subscriber joins: late subscriber gets `next(1) next(2)` from the buffer if bufferSize allows.

## Why this is Mealy rather than Moore

Join input replays state. Source next writes the new letter. Buffer size and window are parameters of `T`.

## Edge cases fixed by the 7.x source

- Does not refCount unless composed with `refCount`.
- Deprecated in favor of `share({ connector: () => new ReplaySubject(...) })`.

## Source anchors

- `src/internal/operators/publishReplay.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
