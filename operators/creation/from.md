# `from` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold conversion function |
| RxJS 7.x source | `src/internal/observable/from.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `from(input: ObservableInput<T>, scheduler?: SchedulerLike): Observable<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`from` is a cold conversion function on the RxJS 7.x line. Stable. `from` adapts an `ObservableInput` into an Observable. Arrays and array-likes are enumerated; iterables are pulled; promises become a single next+complete or an error; an existing Observable is subscribed and forwarded; async iterables are pulled until return. The scheduler, if any, spreads synchronous emissions.

In plain terms, the operator keeps this memory: S = { idle, pulling(cursor), awaiting(promise | asyncIterator), forwarding, stopped }. The cursor is the iteration state. Promise and async-iterator waits are distinct only in how the next input is produced. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, yield(v), yieldDone, reject(e), innerNext, innerError, innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: from([10, 20]): subscribe, yield(10), yield(20), yieldDone writes next(10) next(20) complete.

Details that a marble diagram often leaves out: A string is an iterable of characters. Readable-stream style async iterables must be closed on unsubscribe. Scheduler does not change the word, only when its letters are delivered.

## Role in the notification machine

`from` adapts an `ObservableInput` into an Observable. Arrays and array-likes are enumerated; iterables are pulled; promises become a single next+complete or an error; an existing Observable is subscribed and forwarded; async iterables are pulled until return. The scheduler, if any, spreads synchronous emissions.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, pulling(cursor), awaiting(promise | asyncIterator), forwarding, stopped }`. The cursor is the iteration state. Promise and async-iterator waits are distinct only in how the next input is produced.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, yield(v), yieldDone, reject(e), innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → pulling(0)` for array/iterable, `awaiting` for promise/async iterable, `forwarding` for an Observable, `stopped` if conversion throws.
- `pulling(i) × yield(v) → pulling(i+1)`.
- `pulling × yieldDone → stopped`.
- `awaiting × yield(v) → awaiting` (async iterator) or `stopped` (promise success is terminal).
- `awaiting × reject(e) → stopped`.
- `forwarding` mirrors the inner terminal transitions.
- `unsubscribe → stopped` (async iterator `return()` is attempted).

## 6. Output function (G : S × Z → A*)

- `pulling × yield(v) → next(v)`.
- `pulling × yieldDone → complete`.
- `awaiting × yield(v) → next(v)` and, for a promise, also `· complete`.
- `reject(e) → error(e)`.
- Observable input: identity forward.

## Worked trace

`from([10, 20])`: subscribe, `yield(10)`, `yield(20)`, `yieldDone` writes `next(10) next(20) complete`.

## Why this is Mealy rather than Moore

The same `pulling` state writes `next` or `complete` according to whether the iterator input is a value or done.

## Edge cases fixed by the 7.x source

- A string is an iterable of characters.
- Readable-stream style async iterables must be closed on unsubscribe.
- Scheduler does not change the word, only when its letters are delivered.

## Source anchors

- `src/internal/observable/from.ts` dispatches on the input kind.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
