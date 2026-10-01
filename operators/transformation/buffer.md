# `buffer` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/buffer.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `buffer(closingNotifier: Observable<any>): OperatorFunction<T, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`buffer` is a pipeable operator on the RxJS 7.x line. Stable. Collect source values into an array. Each time `closingNotifier` emits, emit the current array and start a fresh one. Notifier error errors the output. Source error errors the output. Source complete emits the open buffer then completes. Notifier complete emits the open buffer and completes.

In plain terms, the operator keeps this memory: S = { open(buf), stopped }. buf is the array collected since the last close. At subscription, before any source notification, that memory is open([]). It reacts to these events: { next(v), error(e), complete, notifierNext, notifierError, notifierComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source 1, 2, notifier click, source 3, source complete writes next([1, 2]) next([3]) complete.

Details that a marble diagram often leaves out: A notifier next with an empty buffer still emits `[]`. Values are not shared across buffers; the array is replaced.

## Role in the notification machine

Collect source values into an array. Each time `closingNotifier` emits, emit the current array and start a fresh one. Notifier error errors the output. Source error errors the output. Source complete emits the open buffer then completes. Notifier complete emits the open buffer and completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { open(buf), stopped }`. `buf` is the array collected since the last close.

## 2. Initial state (S0)

`open([])`.

## 3. Input alphabet (Z)

`{ next(v), error(e), complete, notifierNext, notifierError, notifierComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T[]), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `open(buf) × next(v) → open(buf ++ [v])`.
- `open(buf) × notifierNext → open([])`.
- `open × (complete | notifierComplete | error | notifierError | unsubscribe) → stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → ε` (value is stored, not forwarded).
- `notifierNext → next(buf)` using the buffer from before the reset.
- `complete` or `notifierComplete → next(buf) · complete`.
- `error(e)` or `notifierError(e) → error(e)`.

## Worked trace

Source `1, 2`, notifier click, source `3`, source complete writes `next([1, 2]) next([3]) complete`.

## Why this is Mealy rather than Moore

`notifierNext` writes the buffer currently in state. The same `open` state writes `ε` on source next and `next(buf)` on notifier next.

## Edge cases fixed by the 7.x source

- A notifier next with an empty buffer still emits `[]`.
- Values are not shared across buffers; the array is replaced.

## Source anchors

- `src/internal/operators/buffer.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
