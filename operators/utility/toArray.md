# `toArray` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/toArray.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `toArray(): OperatorFunction<T, T[]>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`toArray` is a pipeable operator on the RxJS 7.x line. Stable. Buffer every source next. On complete, emit one array and complete. Error passes through and drops the buffer. Does not emit on an infinite source.

In plain terms, the operator keeps this memory: S = { collecting(buf), stopped }. At subscription, before any source notification, that memory is collecting([]). It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2, 3).pipe(toArray()) writes next([1, 2, 3]) complete.

Details that a marble diagram often leaves out: Empty source emits `next([]) complete`. Unbounded memory.

## Role in the notification machine

Buffer every source next. On complete, emit one array and complete. Error passes through and drops the buffer. Does not emit on an infinite source.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { collecting(buf), stopped }`.

## 2. Initial state (S0)

`collecting([])`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(T[]), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` appends.
- `complete → stopped`.
- `error → stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `complete → next(buf) · complete`.
- `error → error`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { collecting(buf), stopped }.

Memory at subscribe: collecting([]).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next(v) appends | next(v) appends | ε |
| complete | stopped | next(buf) · complete |
| error | stopped | error |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1, 2, 3).pipe(toArray())` writes `next([1, 2, 3]) complete`.

## Why this is Mealy rather than Moore

Complete input expands buffer state into a single next. Value inputs write `ε`.

## Edge cases fixed by the 7.x source

- Empty source emits `next([]) complete`.
- Unbounded memory.

## Source anchors

- `src/internal/operators/toArray.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
