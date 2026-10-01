# `bufferToggle` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/bufferToggle.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `bufferToggle(openings, closingSelector): OperatorFunction<T, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`bufferToggle` is a pipeable operator on the RxJS 7.x line. Stable. `openings` emits start signals. Each opening subscribes to `closingSelector(openingValue)`. Source values are copied into every buffer opened and not yet closed. A closing emission emits that buffer and drops it.

In plain terms, the operator keeps this memory: S = { list of { buf, closingSub }, stopped }. At subscription, before any source notification, that memory is No open buffers. It reacts to these events: { next(v), error, complete, opening(o), openingError, closing_i, closingError_i, closingComplete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Opening at value 1, values 1 2, closing, writes next([1, 2]). Values that arrive with no open buffer are dropped.

Details that a marble diagram often leaves out: Source complete does not emit partial buffers. Multiple openings can overlap.

## Role in the notification machine

`openings` emits start signals. Each opening subscribes to `closingSelector(openingValue)`. Source values are copied into every buffer opened and not yet closed. A closing emission emits that buffer and drops it.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { list of { buf, closingSub }, stopped }`.

## 2. Initial state (S0)

No open buffers.

## 3. Input alphabet (Z)

`{ next(v), error, complete, opening(o), openingError, closing_i, closingError_i, closingComplete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T[]), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `opening(o)` appends a new buffer and subscribes to its closer.
- `next(v)` appends to all open buffers.
- `closing_i` removes buffer i.
- Any error → `stopped`. Source complete → `stopped` (open buffers are not flushed by source complete in bufferToggle; they are discarded).
- Closing complete without a next does not emit; it just ends that closer.

## 6. Output function (G : S × Z → A*)

- `closing_i → next(buf_i)`.
- `next(v) → ε`.
- `error → error`.
- `complete → complete` with no trailing buffers.

## Worked trace

Opening at value 1, values 1 2, closing, writes `next([1, 2])`. Values that arrive with no open buffer are dropped.

## Why this is Mealy rather than Moore

The closing input selects which buffer state to write. Source next writes `ε` even though it changes state.

## Edge cases fixed by the 7.x source

- Source complete does not emit partial buffers.
- Multiple openings can overlap.

## Source anchors

- `src/internal/operators/bufferToggle.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
