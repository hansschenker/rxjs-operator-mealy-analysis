# `sequenceEqual` — Mealy 6-tuple

| | |
|---|---|
| Category | Conditional and Boolean |
| Kind | Pipeable comparison |
| RxJS 7.x source | `src/internal/operators/sequenceEqual.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `sequenceEqual(compareTo, comparator?): OperatorFunction<T, boolean>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`sequenceEqual` is a pipeable comparison on the RxJS 7.x line. Stable. Compare the source to `compareTo` index by index, like zip plus an equality check. Emit `false` and complete on the first mismatch or if the lengths differ. Emit `true` and complete if both complete with equal paired values. Comparator defaults to `===`.

In plain terms, the operator keeps this memory: Two queues plus done flags, or stopped. Buffers values that arrive before their pair. At subscription, before any source notification, that memory is Empty queues, neither done. It reacts to these events: { next_source, next_other, error_either, complete_source, complete_other, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2).pipe(sequenceEqual(of(1, 2))) writes next(true) complete. Against of(1, 3) it writes next(false) complete at the second pair.

Details that a marble diagram often leaves out: Uses a comparator, not deep equality. Errors from either side are forwarded.

## Role in the notification machine

Compare the source to `compareTo` index by index, like zip plus an equality check. Emit `false` and complete on the first mismatch or if the lengths differ. Emit `true` and complete if both complete with equal paired values. Comparator defaults to `===`.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

Two queues plus done flags, or stopped. Buffers values that arrive before their pair.

## 2. Initial state (S0)

Empty queues, neither done.

## 3. Input alphabet (Z)

`{ next_source, next_other, error_either, complete_source, complete_other, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(boolean), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- A next appends to that side's queue. When both queues are non-empty, shift a pair.
- A length mismatch on complete → stopped.
- Both done with empty queues → stopped.

## 6. Output function (G : S × Z → A*)

- A shifted pair that compares equal → ε.
- A shifted pair that compares unequal → next(false) · complete.
- Complete that reveals a leftover value on the other side → next(false) · complete.
- Both exhausted equally → next(true) · complete.

## Worked trace

`of(1, 2).pipe(sequenceEqual(of(1, 2)))` writes `next(true) complete`. Against `of(1, 3)` it writes `next(false) complete` at the second pair.

## Why this is Mealy rather than Moore

The comparison input is a pair formed from state and the arriving next. Equal and unequal pairs write different words from the same buffering machine.

## Edge cases fixed by the 7.x source

- Uses a comparator, not deep equality.
- Errors from either side are forwarded.

## Source anchors

- `src/internal/operators/sequenceEqual.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
