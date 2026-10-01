# `shareReplay` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable replay multicast |
| RxJS 7.x source | `src/internal/operators/shareReplay.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `shareReplay(configOrBufferSize?, windowTime?, scheduler?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`shareReplay` is a pipeable replay multicast on the RxJS 7.x line. Stable. `share` with a ReplaySubject connector. On 7.x the implementation sets `resetOnError: true`, `resetOnComplete: false`, and `resetOnRefCountZero` from the `refCount` option (default false). So the replay buffer survives completion and, by default, survives the last subscriber leaving. New subscribers receive the buffered window.

In plain terms, the operator keeps this memory: { cold, hot(buffer, refCount), stopped } with a ReplaySubject buffer. At subscription, before any source notification, that memory is cold, empty buffer. It reacts to these events: { subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete, resetTick }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: First subscriber sees 1, 2 and unsubscribes. A later subscriber still receives 1, 2 from the buffer if refCount is false and the source already produced them.

Details that a marble diagram often leaves out: Default refCount false keeps the source subscribed. Config object form accepts bufferSize, windowTime, refCount, scheduler. Implemented via `share`.

## Role in the notification machine

`share` with a ReplaySubject connector. On 7.x the implementation sets `resetOnError: true`, `resetOnComplete: false`, and `resetOnRefCountZero` from the `refCount` option (default false). So the replay buffer survives completion and, by default, survives the last subscriber leaving. New subscribers receive the buffered window.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ cold, hot(buffer, refCount), stopped }` with a ReplaySubject buffer.

## 2. Initial state (S0)

`cold`, empty buffer.

## 3. Input alphabet (Z)

`{ subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete, resetTick }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- First join connects and subscribes to the source.
- Source next appends to the replay buffer.
- Source complete does not reset (default).
- Source error resets because resetOnError is true.
- Leave at refCount 0 does not unsubscribe the source unless refCount: true.

## 6. Output function (G : S × Z → A*)

- Join → next for each buffered value, then live values.
- Source next → next to current subscribers and a buffer append.
- Source complete → complete, and late join still replays then completes.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { cold, hot(buffer, refCount), stopped } with a ReplaySubject buffer.

Memory at subscribe: cold, empty buffer.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| First join connects and subscribes to the source | First join connects and subscribes to the source | next to current subscribers and a buffer append |
| Source next appends to the replay buffer | Source next appends to the replay buffer | next for each buffered value, then live values |
| Source complete does not reset (default) | Source complete does not reset (default) | complete, and late join still replays then completes |
| Source error resets because resetOnError is true | Source error resets because resetOnError is true | nothing named on a separate output row |
| Leave at refCount 0 does not unsubscribe the source unless refCount: true | Leave at refCount 0 does not unsubscribe the source unless refCount: true | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

First subscriber sees 1, 2 and unsubscribes. A later subscriber still receives 1, 2 from the buffer if refCount is false and the source already produced them.

## Why this is Mealy rather than Moore

Join writes a word taken from buffer state. Source next writes the new letter and updates that buffer. Reset flags change T, not the letter shape.

## Edge cases fixed by the 7.x source

- Default refCount false keeps the source subscribed.
- Config object form accepts bufferSize, windowTime, refCount, scheduler.
- Implemented via `share`.

## Source anchors

- `src/internal/operators/shareReplay.ts` calls `share` with a ReplaySubject connector.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
