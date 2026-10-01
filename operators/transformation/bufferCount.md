# `bufferCount` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferCount.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `bufferCount(bufferSize: number, startBufferEvery: number = bufferSize): OperatorFunction<T, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`bufferCount` is a pipeable operator on the RxJS 7.x line. Stable. Maintain a list of open buffers. Every `startBufferEvery` source values, open a new buffer. A buffer emits and closes when it reaches `bufferSize`. Default `startBufferEvery = bufferSize` is non-overlapping. On complete, emit every non-empty open buffer then complete.

In plain terms, the operator keeps this memory: S = { buffers: T[][], count since last open, stopped }. At subscription, before any source notification, that memory is One empty buffer, count 0, unless bufferSize < 1 which errors. It reacts to these events: { next(v), error(e), complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: bufferCount(3, 1) on a b c d emits [a,b,c], then [b,c,d], and on complete the trailing partials [c,d] and [d].

Details that a marble diagram often leaves out: `bufferSize < 1` errors on subscribe. `startBufferEvery` smaller than `bufferSize` creates overlapping buffers.

## Role in the notification machine

Maintain a list of open buffers. Every `startBufferEvery` source values, open a new buffer. A buffer emits and closes when it reaches `bufferSize`. Default `startBufferEvery = bufferSize` is non-overlapping. On complete, emit every non-empty open buffer then complete.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { buffers: T[][], count since last open, stopped }`.

## 2. Initial state (S0)

One empty buffer, count 0, unless `bufferSize < 1` which errors.

## 3. Input alphabet (Z)

`{ next(v), error(e), complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T[]), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` appends `v` to every open buffer, may close full buffers, and may open a new buffer when `count` hits `startBufferEvery`.
- Terminal inputs → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) →` the concatenation of `next(buf)` for each buffer that just reached `bufferSize`, else `ε`.
- `complete → next(buf)` for each non-empty open buffer, then `complete`.
- `error(e) → error(e)`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { buffers: T[][], count since last open, stopped }.

Memory at subscribe: One empty buffer, count 0, unless bufferSize < 1 which errors.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next(v) appends v to every open buffer, may close full buffers, and may open a new buffer when count hits startBufferEvery | next(v) appends v to every open buffer, may close full buffers, and may open a new buffer when count hits startBufferEvery | the concatenation of next(buf) for each buffer that just reached bufferSize, else ε |
| Terminal inputs | stopped | next(buf) for each non-empty open buffer, then complete |
| error(e) | named by the output row; memory change is in the transition rows above | error(e) |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`bufferCount(3, 1)` on `a b c d` emits `[a,b,c]`, then `[b,c,d]`, and on complete the trailing partials `[c,d]` and `[d]`.

## Why this is Mealy rather than Moore

Whether a source next writes a buffer word depends on the length state of each open buffer. Overlap is encoded as multiple buffers in `S`.

## Edge cases fixed by the 7.x source

- `bufferSize < 1` errors on subscribe.
- `startBufferEvery` smaller than `bufferSize` creates overlapping buffers.

## Source anchors

- `src/internal/operators/bufferCount.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
