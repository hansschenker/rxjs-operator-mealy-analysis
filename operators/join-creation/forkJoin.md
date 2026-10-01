# `forkJoin` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/forkJoin.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `forkJoin(sources): Observable<T[] | Record<K, V>>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`forkJoin` is a join creation function on the RxJS 7.x line. Stable. Subscribe to all sources. Keep only the last value of each. Emit once, when every source has completed, the array or dictionary of last values, then complete. If any source errors, error and unsubscribe the rest. If any source completes without a value, complete without emitting.

In plain terms, the operator keeps this memory: S = per-source { last: V | ⊥, done: bool } plus stopped. At subscription, before any source notification, that memory is All last = ⊥, done = false. It reacts to these events: { subscribe, next_i(v), error_i, complete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: forkJoin([of(1, 2), of('a')]) writes a single next([2, 'a']) complete after both complete. The 1 is overwritten and never emitted.

Details that a marble diagram often leaves out: Empty argument list completes without a next. Dictionary keys are preserved. Unlike `combineLatest`, intermediate snapshots are not outputs.

## Role in the notification machine

Subscribe to all sources. Keep only the last value of each. Emit once, when every source has completed, the array or dictionary of last values, then complete. If any source errors, error and unsubscribe the rest. If any source completes without a value, complete without emitting.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = per-source { last: V | ⊥, done: bool } plus `stopped`.

## 2. Initial state (S0)

All `last = ⊥`, `done = false`.

## 3. Input alphabet (Z)

`{ subscribe, next_i(v), error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(lasts), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next_i(v)` overwrites `last_i`.
- `complete_i` sets `done_i`. All done → `stopped`.
- `error_i → stopped`.

## 6. Output function (G : S × Z → A*)

- `next_i → ε` always (forkJoin does not emit on next).
- `complete_i → next(lasts) · complete` if every source is done and every source has a last value.
- `complete_i → complete` if every source is done but some last is ⊥.
- `error_i(e) → error(e)`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = per-source { last: V \| ⊥, done: bool } plus stopped.

Memory at subscribe: All last = ⊥, done = false.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next_i(v) overwrites last_i | next_i(v) overwrites last_i | ε always (forkJoin does not emit on next) |
| complete_i | stopped | next(lasts) · complete if every source is done and every source has a last value |
| error_i | stopped | error(e) |
| complete_i | named by the output row; memory change is in the transition rows above | complete if every source is done but some last is ⊥ |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`forkJoin([of(1, 2), of('a')])` writes a single `next([2, 'a']) complete` after both complete. The `1` is overwritten and never emitted.

## Why this is Mealy rather than Moore

The complete input is what triggers the output word, and the word's payload is the state. Classic Mealy: completion is the input, the tuple is memory.

## Edge cases fixed by the 7.x source

- Empty argument list completes without a next.
- Dictionary keys are preserved.
- Unlike `combineLatest`, intermediate snapshots are not outputs.

## Source anchors

- `src/internal/observable/forkJoin.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
