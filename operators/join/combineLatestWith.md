# `combineLatestWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable join |
| RxJS 7.x source | `src/internal/operators/combineLatestWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `combineLatestWith(...otherSources): OperatorFunction<T, [T, ...A]>` |
| Status on the 7.x line | Stable. Pipeable replacement for the old `combineLatest` operator signature. |

SuperGrok is the main contributor of this analysis.

## Explanation

`combineLatestWith` is a pipeable join on the RxJS 7.x line. Stable. Pipeable replacement for the old `combineLatest` operator signature. Subscribe to the source and to each other source. Remember the latest of each. Emit a tuple only after every participant has a value, then on every subsequent next from any of them. Complete when all complete. Error if any errors.

In plain terms, the operator keeps this memory: Per-source { latest: V | ⊥, done } plus stopped. Index 0 is the piped source. At subscription, before any source notification, that memory is All latest ⊥, none done. It reacts to these events: { next_i, error_i, complete_i, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source emits 1 before the other emits: ε. Other emits a: next([1, a]). Source emits 2: next([2, a]).

Details that a marble diagram often leaves out: Equivalent to `combineLatest([source, ...others])` after subscription. Does not emit on complete by itself.

## Role in the notification machine

Subscribe to the source and to each other source. Remember the latest of each. Emit a tuple only after every participant has a value, then on every subsequent next from any of them. Complete when all complete. Error if any errors.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Per-source `{ latest: V | ⊥, done }` plus stopped. Index 0 is the piped source.

## 2. Initial state (S0)

All latest `⊥`, none done.

## 3. Input alphabet (Z)

`{ next_i, error_i, complete_i, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(tuple), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next_i` stores latest_i.
- All done → stopped.
- Any error → stopped.

## 6. Output function (G : S × Z → A*)

- `next_i → next(snapshot)` if every latest is present, else `ε`.
- `complete_i → complete` only when all are done.
- `error_i → error`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: Per-source { latest: V \| ⊥, done } plus stopped. Index 0 is the piped source.

Memory at subscribe: All latest ⊥, none done.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next_i stores latest_i | next_i stores latest_i | next(snapshot) if every latest is present, else ε |
| All done | stopped | complete only when all are done |
| Any error | stopped | error |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Source emits 1 before the other emits: `ε`. Other emits `a`: `next([1, a])`. Source emits 2: `next([2, a])`.

## Why this is Mealy rather than Moore

The same next input writes `ε` or a tuple depending on whether the other slots in state are filled.

## Edge cases fixed by the 7.x source

- Equivalent to `combineLatest([source, ...others])` after subscription.
- Does not emit on complete by itself.

## Source anchors

- `src/internal/operators/combineLatestWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
