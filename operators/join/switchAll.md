# `switchAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/switchAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `switchAll(): OperatorFunction<ObservableInput<T>, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`switchAll` is a pipeable higher-order join on the RxJS 7.x line. Stable. Subscribe to each new inner and unsubscribe the previous one. Only the latest inner's values are forwarded.

In plain terms, the operator keeps this memory: S = { active inner | ⊥, outerDone, stopped }. At subscription, before any source notification, that memory is No inner. It reacts to these events: Higher-order alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source emitting inner A then inner B unsubscribes A; only B's values appear after the switch.

Details that a marble diagram often leaves out: No queue. A sync inner can emit before the next outer next switches it.

## Role in the notification machine

Subscribe to each new inner and unsubscribe the previous one. Only the latest inner's values are forwarded.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active inner | ⊥, outerDone, stopped }`.

## 2. Initial state (S0)

No inner.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` replaces active.
- `innerComplete` clears active.
- Outer done and idle → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- Previous inner's later signals are not delivered.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { active inner \| ⊥, outerDone, stopped }.

Memory at subscribe: No inner.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| outerNext replaces active | outerNext replaces active | next |
| innerComplete clears active | innerComplete clears active | nothing named on a separate output row |
| Outer done and idle | stopped | Previous inner's later signals are not delivered |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Source emitting inner A then inner B unsubscribes A; only B's values appear after the switch.

## Why this is Mealy rather than Moore

Same machine as `switchMap` with identity project.

## Edge cases fixed by the 7.x source

- No queue.
- A sync inner can emit before the next outer next switches it.

## Source anchors

- `src/internal/operators/switchAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
