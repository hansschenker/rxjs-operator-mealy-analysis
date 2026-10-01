# `auditTime` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/auditTime.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `auditTime(duration, scheduler = asyncScheduler): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`auditTime` is a pipeable operator on the RxJS 7.x line. Stable. `audit` with a timer duration. The first value in an idle period starts a timer and is not emitted yet. Values during the timer overwrite the pending value. Timer fire emits the latest and goes idle.

In plain terms, the operator keeps this memory: S = { idle, auditing(latest), stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { next(v), error, complete, tick, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two values inside the duration and a tick write a single next of the second value.

Details that a marble diagram often leaves out: Unlike `throttleTime`, this is trailing-edge. Complete flushes.

## Role in the notification machine

`audit` with a timer duration. The first value in an idle period starts a timer and is not emitted yet. Values during the timer overwrite the pending value. Timer fire emits the latest and goes idle.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, auditing(latest), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, tick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × next(v) → auditing(v)`.
- `auditing × next(v) → auditing(v)`.
- `tick → idle`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- Source next → `ε`.
- `tick → next(latest)`.
- `complete` flushes pending then completes.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { idle, auditing(latest), stopped }.

Memory at subscribe: idle.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| idle × next(v) | auditing(v) | ε |
| auditing × next(v) | auditing(v) | next(latest) |
| tick | idle | nothing named on a separate output row |
| Terminal | stopped | complete flushes pending then completes |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Two values inside the duration and a tick write a single `next` of the second value.

## Why this is Mealy rather than Moore

Tick input, not the source next, is what produces the output letter from pending state.

## Edge cases fixed by the 7.x source

- Unlike `throttleTime`, this is trailing-edge.
- Complete flushes.

## Source anchors

- `src/internal/operators/auditTime.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
