# `combineAll` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Deprecated alias |
| RxJS 7.x source | `src/internal/operators/combineAll.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `combineAll(project?): OperatorFunction<ObservableInput<T>, T[]>` |
| Status on the 7.x line | Deprecated alias of `combineLatestAll`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`combineAll` is a deprecated alias on the RxJS 7.x line. Deprecated alias of `combineLatestAll`. `combineAll` is a one-line re-export of `combineLatestAll`. The machine is the higher-order combineLatest machine: collect inners until the outer completes, then emit snapshots once every inner has a latest.

In plain terms, the operator keeps this memory: Same as combineLatestAll: collected inners, latest per inner, outerDone, stopped. At subscription, before any source notification, that memory is No inners collected. It reacts to these events: { outerNext(inner$), outerComplete, outerError, innerNext_i, innerError_i, innerComplete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Same word as combineLatestAll on the same higher-order source.

Details that a marble diagram often leaves out: File is a re-export. Behavior lives in `combineLatestAll.ts`. Removed in later majors.

## Role in the notification machine

`combineAll` is a one-line re-export of `combineLatestAll`. The machine is the higher-order combineLatest machine: collect inners until the outer completes, then emit snapshots once every inner has a latest.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Same as `combineLatestAll`: collected inners, latest per inner, outerDone, stopped.

## 2. Initial state (S0)

No inners collected.

## 3. Input alphabet (Z)

`{ outerNext(inner$), outerComplete, outerError, innerNext_i, innerError_i, innerComplete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Identical to `combineLatestAll`.
- Deprecation does not add a state.

## 6. Output function (G : S × Z → A*)

- Identical to `combineLatestAll`: an inner next writes a snapshot only when every collected inner has a latest.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: Same as combineLatestAll: collected inners, latest per inner, outerDone, stopped.

Memory at subscribe: No inners collected.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Identical to combineLatestAll | Identical to combineLatestAll | Identical to combineLatestAll: an inner next writes a snapshot only when every collected inner has a latest |
| Deprecation does not add a state | Deprecation does not add a state | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Same word as `combineLatestAll` on the same higher-order source.

## Why this is Mealy rather than Moore

The alias does not change the Mealy decision: emit-or-ε depends on readiness state and the arriving inner next.

## Edge cases fixed by the 7.x source

- File is a re-export. Behavior lives in `combineLatestAll.ts`.
- Removed in later majors.

## Source anchors

- `src/internal/operators/combineAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
