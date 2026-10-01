# `partition` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Splitting function, not a pipeable operator |
| RxJS 7.x source | `src/internal/operators/partition.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `partition(source, predicate, thisArg?): [Observable<T>, Observable<T>]` |
| Status on the 7.x line | Stable function. Listed by the docs under both join creation and transformation. Not used inside `pipe`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`partition` is a splitting function, not a pipeable operator on the RxJS 7.x line. Stable function. Listed by the docs under both join creation and transformation. Not used inside `pipe`. `partition` returns two observables: values for which `predicate` is true, and values for which it is false. On 7.x it is implemented as two `filter` subscriptions, not as one shared multicast. A cold source therefore runs twice, once per branch, unless the caller `share`s it first. The Mealy model below is the logical splitter; the source note records the double subscription.

In plain terms, the operator keeps this memory: Logical splitter: S = { active(i), stopped } with i the source index passed to the predicate. Implementation: two independent filter machines. At subscription, before any source notification, that memory is active(0) per branch subscription. It reacts to these events: { next(v), error(e), complete, unsubscribe } on each subscription.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: partition(of(1,2,3), x => x % 2) yields pass word next(1) next(3) complete and fail word next(2) complete, but of is subscribed twice.

Details that a marble diagram often leaves out: Not a single subscription. Side-effecting sources run per branch. Index is the index in that branch's subscription, which coincide only if both are subscribed and the source is deterministic. `thisArg` is deprecated style.

## Role in the notification machine

`partition` returns two observables: values for which `predicate` is true, and values for which it is false. On 7.x it is implemented as two `filter` subscriptions, not as one shared multicast. A cold source therefore runs twice, once per branch, unless the caller `share`s it first. The Mealy model below is the logical splitter; the source note records the double subscription.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Logical splitter: `S = { active(i), stopped }` with `i` the source index passed to the predicate. Implementation: two independent filter machines.

## 2. Initial state (S0)

`active(0)` per branch subscription.

## 3. Input alphabet (Z)

`{ next(v), error(e), complete, unsubscribe }` on each subscription.

## 4. Output alphabet (A)

Branch A: `{ next(v) if predicate, error, complete }`. Branch B: the negation.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Each branch: `active(i) × next(v) → active(i+1)`.
- Terminal inputs → `stopped` on that branch only.

## 6. Output function (G : S × Z → A*)

- Pass branch: `next(v) → next(v)` if `predicate(v, i)`, else `ε`.
- Fail branch: `next(v) → next(v)` if not predicate, else `ε`.
- Error and complete are copied to the branch that received them.

## Worked trace

`partition(of(1,2,3), x => x % 2)` yields pass word `next(1) next(3) complete` and fail word `next(2) complete`, but `of` is subscribed twice.

## Why this is Mealy rather than Moore

The predicate decision is exactly `G(active(i), next(v))`, a Mealy output. The index in state is part of the predicate input.

## Edge cases fixed by the 7.x source

- Not a single subscription. Side-effecting sources run per branch.
- Index is the index in that branch's subscription, which coincide only if both are subscribed and the source is deterministic.
- `thisArg` is deprecated style.

## Source anchors

- `src/internal/operators/partition.ts` builds `[filter(predicate)(source), filter(notPredicate)(source)]`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
