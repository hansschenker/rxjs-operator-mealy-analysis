# `bufferTime` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferTime.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `bufferTime(bufferTimeSpan, bufferCreationInterval?, maxBufferSize?, scheduler?): OperatorFunction<T, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`bufferTime` is a pipeable operator on the RxJS 7.x line. Stable. Open a buffer and emit it when `bufferTimeSpan` elapses. Optional `bufferCreationInterval` opens additional buffers on a cadence (overlapping windows). Optional `maxBufferSize` closes a buffer early when it fills. Scheduler defaults to async.

In plain terms, the operator keeps this memory: S = { open buffers with their close times, stopped }. At subscription, before any source notification, that memory is One open buffer armed to close after bufferTimeSpan. It reacts to these events: { next(v), error, complete, spanTick(id), creationTick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: bufferTime(1000) over values inside one second emits one array per second, including empty arrays when a span had no values.

Details that a marble diagram often leaves out: Empty time spans still emit `[]`. Creation interval plus span is the overlapping form.

## Role in the notification machine

Open a buffer and emit it when `bufferTimeSpan` elapses. Optional `bufferCreationInterval` opens additional buffers on a cadence (overlapping windows). Optional `maxBufferSize` closes a buffer early when it fills. Scheduler defaults to async.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { open buffers with their close times, stopped }`.

## 2. Initial state (S0)

One open buffer armed to close after `bufferTimeSpan`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, spanTick(id), creationTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T[]), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` appends to open buffers; a buffer at `maxBufferSize` closes.
- `spanTick(id)` closes that buffer and, if no creation interval, opens a successor.
- `creationTick` opens another buffer.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `spanTick` and max-size close → `next(buf)`.
- `complete →` emit open buffers then `complete`.
- `next` itself → `ε` unless it hit `maxBufferSize`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { open buffers with their close times, stopped }.

Memory at subscribe: One open buffer armed to close after bufferTimeSpan.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next(v) appends to open buffers; a buffer at maxBufferSize closes | next(v) appends to open buffers; a buffer at maxBufferSize closes | next(buf) |
| spanTick(id) closes that buffer and, if no creation interval, opens a successor | spanTick(id) closes that buffer and, if no creation interval, opens a successor | nothing named on a separate output row |
| creationTick opens another buffer | creationTick opens another buffer | nothing named on a separate output row |
| Terminal | stopped | emit open buffers then complete |
| next itself | named by the output row; memory change is in the transition rows above | ε unless it hit maxBufferSize |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`bufferTime(1000)` over values inside one second emits one array per second, including empty arrays when a span had no values.

## Why this is Mealy rather than Moore

Time ticks are inputs. The emitted array is the state of that buffer. A tick and a source next in the same control regime write different words.

## Edge cases fixed by the 7.x source

- Empty time spans still emit `[]`.
- Creation interval plus span is the overlapping form.

## Source anchors

- `src/internal/operators/bufferTime.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
