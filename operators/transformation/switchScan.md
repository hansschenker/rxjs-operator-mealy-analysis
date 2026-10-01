# `switchScan` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order accumulator |
| RxJS 7.x source | `src/internal/operators/switchScan.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `switchScan(accumulator, seed): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`switchScan` is a pipeable higher-order accumulator on the RxJS 7.x line. Stable. Like `mergeScan` with switch semantics. A new outer value unsubscribes the active accumulator inner and subscribes to `accumulator(latestAcc, value)`. Inner emissions update `acc` and are forwarded.

In plain terms, the operator keeps this memory: S = { acc, active inner | ⊥, outerDone, stopped }. At subscription, before any source notification, that memory is acc = seed, no inner. It reacts to these events: Higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A fast outer with a slow accumulator inner: only the latest accumulation survives; previous inner nexts stop.

Details that a marble diagram often leaves out: Seed is required and not emitted up front. Unsubscribed inner emissions are not outputs.

## Role in the notification machine

Like `mergeScan` with switch semantics. A new outer value unsubscribes the active accumulator inner and subscribes to `accumulator(latestAcc, value)`. Inner emissions update `acc` and are forwarded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { acc, active inner | ⊥, outerDone, stopped }`.

## 2. Initial state (S0)

`acc = seed`, no inner.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next(r), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` unsubscribes the active inner and subscribes to the new accumulation.
- `innerNext` replaces `acc`.
- `innerComplete` clears active; if outer done → `stopped`.
- Errors → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext(r) → next(r)`.
- `outerNext → ε` (switch action only).
- `innerComplete → complete` if outer done, else `ε`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { acc, active inner \| ⊥, outerDone, stopped }.

Memory at subscribe: acc = seed, no inner.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| outerNext unsubscribes the active inner and subscribes to the new accumulation | outerNext unsubscribes the active inner and subscribes to the new accumulation | nothing named on a separate output row |
| innerNext replaces acc | innerNext replaces acc | next(r) |
| innerComplete clears active; if outer done | stopped | complete if outer done, else ε |
| Errors | stopped | nothing named on a separate output row |
| outerNext | named by the output row; memory change is in the transition rows above | ε (switch action only) |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

A fast outer with a slow accumulator inner: only the latest accumulation survives; previous inner nexts stop.

## Why this is Mealy rather than Moore

Switch is a transition on outer next that depends on there being an active inner. Emissions still come from inner inputs applied to acc state.

## Edge cases fixed by the 7.x source

- Seed is required and not emitted up front.
- Unsubscribed inner emissions are not outputs.

## Source anchors

- `src/internal/operators/switchScan.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
