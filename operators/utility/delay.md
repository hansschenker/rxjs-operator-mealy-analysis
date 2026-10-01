# `delay` — Mealy 6-tuple

| | |
|---|---|
| Category | Utility |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/delay.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `delay(due: number | Date, scheduler = asyncScheduler): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`delay` is a pipeable operator on the RxJS 7.x line. Stable. Shift next notifications by `due`. Complete is delayed so it stays after delayed nexts. Error is not delayed. Each next is a scheduled action; state is the queue of pending actions.

In plain terms, the operator keeps this memory: S = { pending queue of scheduled nexts, completeArmed, stopped }. At subscription, before any source notification, that memory is Empty queue. It reacts to these events: { next(v), error, complete, dueTick(v), completeTick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1).pipe(delay(1000)) writes nothing at subscribe time and next(1) complete about a second later.

Details that a marble diagram often leaves out: Date due is absolute. Unsubscribe cancels pending ticks.

## Role in the notification machine

Shift next notifications by `due`. Complete is delayed so it stays after delayed nexts. Error is not delayed. Each next is a scheduled action; state is the queue of pending actions.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { pending queue of scheduled nexts, completeArmed, stopped }`.

## 2. Initial state (S0)

Empty queue.

## 3. Input alphabet (Z)

`{ next(v), error, complete, dueTick(v), completeTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next` enqueues a due tick.
- `dueTick` removes that item.
- `complete` arms a complete tick after pending nexts.
- `error → stopped` immediately.

## 6. Output function (G : S × Z → A*)

- `next → ε` (schedule action).
- `dueTick(v) → next(v)`.
- `error → error` now.
- `completeTick → complete`.

## Worked trace

`of(1).pipe(delay(1000))` writes nothing at subscribe time and `next(1) complete` about a second later.

## Why this is Mealy rather than Moore

The due-tick input writes the value that was stored when the source next arrived. Error input writes immediately from the same logical stream.

## Edge cases fixed by the 7.x source

- Date due is absolute.
- Unsubscribe cancels pending ticks.

## Source anchors

- `src/internal/operators/delay.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
