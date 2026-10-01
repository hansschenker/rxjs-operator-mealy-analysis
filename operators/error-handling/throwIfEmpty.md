# `throwIfEmpty` — Mealy 6-tuple

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable guard |
| RxJS 7.x source | `src/internal/operators/throwIfEmpty.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `throwIfEmpty(errorFactory = defaultEmptyErrorFactory): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`throwIfEmpty` is a pipeable guard on the RxJS 7.x line. Stable. Forward nexts. If the source completes without a next, error with the factory result (EmptyError by default) instead of completing. One seen-flag of memory.

In plain terms, the operator keeps this memory: { empty, seen, stopped }. At subscription, before any source notification, that memory is empty. It reacts to these events: { next, error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: EMPTY.pipe(throwIfEmpty()) writes error(EmptyError). of(1).pipe(throwIfEmpty()) writes next(1) complete.

Details that a marble diagram often leaves out: Factory runs only on the empty-complete path. Used internally by operators that must reject an empty source.

## Role in the notification machine

Forward nexts. If the source completes without a next, error with the factory result (EmptyError by default) instead of completing. One seen-flag of memory.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ empty, seen, stopped }`.

## 2. Initial state (S0)

`empty`.

## 3. Input alphabet (Z)

`{ next, error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Next → seen.
- Complete → stopped.
- Error → stopped.

## 6. Output function (G : S × Z → A*)

- Next → next.
- Complete in empty → error(factory()).
- Complete in seen → complete.
- Error copies through.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { empty, seen, stopped }.

Memory at subscribe: empty.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Next | seen | next |
| Complete | stopped | error(factory()) |
| Error | stopped | Error copies through |
| Complete in seen | named by the output row; memory change is in the transition rows above | complete |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`EMPTY.pipe(throwIfEmpty())` writes `error(EmptyError)`. `of(1).pipe(throwIfEmpty())` writes `next(1) complete`.

## Why this is Mealy rather than Moore

Complete writes either error or complete depending on the seen flag. Same input symbol, different word.

## Edge cases fixed by the 7.x source

- Factory runs only on the empty-complete path.
- Used internally by operators that must reject an empty source.

## Source anchors

- `src/internal/operators/throwIfEmpty.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
