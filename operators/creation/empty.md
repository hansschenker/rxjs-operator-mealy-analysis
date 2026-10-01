# `empty` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function / constant |
| RxJS 7.x source | `src/internal/observable/empty.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `empty(scheduler?: SchedulerLike): Observable<never>` |
| Status on the 7.x line | Deprecated on 7.x in favor of the `EMPTY` constant. Still present. |

SuperGrok is the main contributor of this analysis.

## Explanation

`empty` is a cold creation function / constant on the RxJS 7.x line. Deprecated on 7.x in favor of the `EMPTY` constant. Still present. The machine emits no values. Subscribe writes `complete` and stops. An optional scheduler only delays that single complete notification; it does not add values.

In plain terms, the operator keeps this memory: S = { idle, scheduled, stopped }. Without a scheduler, scheduled is skipped. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, schedulerTick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: empty().subscribe(observer) produces the word complete and nothing else.

Details that a marble diagram often leaves out: `EMPTY` is the same machine with no scheduler argument. Deprecated status does not change the tuple.

## Role in the notification machine

The machine emits no values. Subscribe writes `complete` and stops. An optional scheduler only delays that single complete notification; it does not add values.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, scheduled, stopped }`. Without a scheduler, `scheduled` is skipped.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, schedulerTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → stopped` if no scheduler.
- `idle × subscribe → scheduled` if a scheduler is given.
- `scheduled × schedulerTick → stopped`.
- `scheduled × unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `idle × subscribe → complete` (synchronous path).
- `scheduled × schedulerTick → complete`.
- Otherwise `ε`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { idle, scheduled, stopped }. Without a scheduler, scheduled is skipped.

Memory at subscribe: idle.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| idle × subscribe | stopped if no scheduler | complete (synchronous path) |
| idle × subscribe | scheduled if a scheduler is given | nothing named on a separate output row |
| scheduled × schedulerTick | stopped | complete |
| scheduled × unsubscribe | stopped | nothing named on a separate output row |
| Otherwise ε | named by the output row; memory change is in the transition rows above | Otherwise ε |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`empty().subscribe(observer)` produces the word `complete` and nothing else.

## Why this is Mealy rather than Moore

Trivial Mealy: the only productive input is subscribe (or the scheduled tick that subscribe armed). Output is determined by that input, not by sitting in a state.

## Edge cases fixed by the 7.x source

- `EMPTY` is the same machine with no scheduler argument.
- Deprecated status does not change the tuple.

## Source anchors

- `src/internal/observable/empty.ts` exports the creation function; `EMPTY` is the constant.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
