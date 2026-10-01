# `pairs` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Deprecated object enumerator |
| RxJS 7.x source | `src/internal/observable/pairs.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `pairs(obj, scheduler?): Observable<[string, T]>` |
| Status on the 7.x line | Deprecated. Use `from(Object.entries(obj))`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`pairs` is a deprecated object enumerator on the RxJS 7.x line. Deprecated. Use `from(Object.entries(obj))`. On subscribe, enumerate own enumerable keys of `obj` and emit `[key, value]` pairs, then complete. Scheduler spreads the emissions.

In plain terms, the operator keeps this memory: { pending(i), stopped } over the entry list captured at subscribe. At subscription, before any source notification, that memory is pending(0) after the entry list is built. It reacts to these events: { subscribe, step, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: pairs({ a: 1, b: 2 }) writes next(['a', 1]) next(['b', 2]) complete in enumeration order.

Details that a marble diagram often leaves out: Deprecated. Entries are read at subscribe, not when `pairs` is called, so a mutated object is seen per subscription.

## Role in the notification machine

On subscribe, enumerate own enumerable keys of `obj` and emit `[key, value]` pairs, then complete. Scheduler spreads the emissions.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ pending(i), stopped }` over the entry list captured at subscribe.

## 2. Initial state (S0)

`pending(0)` after the entry list is built.

## 3. Input alphabet (Z)

`{ subscribe, step, unsubscribe }`.

## 4. Output alphabet (A)

`{ next([key, value]), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Step advances i.
- Past the last entry → stopped.
- Unsubscribe → stopped.

## 6. Output function (G : S × Z → A*)

- Step at i < n → next(entries[i]).
- Step at n → complete.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { pending(i), stopped } over the entry list captured at subscribe.

Memory at subscribe: pending(0) after the entry list is built.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Step advances i | Step advances i | next(entries[i]) |
| Past the last entry | stopped | complete |
| Unsubscribe | stopped | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`pairs({ a: 1, b: 2 })` writes `next(['a', 1]) next(['b', 2]) complete` in enumeration order.

## Why this is Mealy rather than Moore

The step input writes the entry stored at the index state, the same shape as `of` and `range`.

## Edge cases fixed by the 7.x source

- Deprecated.
- Entries are read at subscribe, not when `pairs` is called, so a mutated object is seen per subscription.

## Source anchors

- `src/internal/observable/pairs.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
