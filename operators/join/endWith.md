# `endWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Join |
| Kind | Pipeable suffix |
| RxJS 7.x source | `src/internal/operators/endWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `endWith(...values, scheduler?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`endWith` is a pipeable suffix on the RxJS 7.x line. Stable. Forward the source. On source complete, emit the given suffix values and then complete. An error skips the suffix. A scheduler shifts the suffix.

In plain terms, the operator keeps this memory: { forwarding, suffix(i), stopped }. At subscription, before any source notification, that memory is forwarding. It reacts to these events: { next, error, complete, suffixStep, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1).pipe(endWith(2, 3)) writes next(1) next(2) next(3) complete.

Details that a marble diagram often leaves out: Suffix is not emitted if the source errors. A scheduler argument is not a suffix value.

## Role in the notification machine

Forward the source. On source complete, emit the given suffix values and then complete. An error skips the suffix. A scheduler shifts the suffix.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ forwarding, suffix(i), stopped }`.

## 2. Initial state (S0)

`forwarding`.

## 3. Input alphabet (Z)

`{ next, error, complete, suffixStep, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source next stays forwarding.
- Source complete → suffix(0).
- Suffix steps advance i, then stopped.
- Error → stopped without suffix.

## 6. Output function (G : S × Z → A*)

- Source next → next.
- Source error → error.
- Suffix step → next(values[i]).
- Last suffix step is followed by complete.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { forwarding, suffix(i), stopped }.

Memory at subscribe: forwarding.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Source next stays forwarding | Source next stays forwarding | next |
| Source complete | suffix(0) | Last suffix step is followed by complete |
| Suffix steps advance i, then stopped | Suffix steps advance i, then stopped | error |
| Error | stopped without suffix | nothing named on a separate output row |
| Suffix step | named by the output row; memory change is in the transition rows above | next(values[i]) |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1).pipe(endWith(2, 3))` writes `next(1) next(2) next(3) complete`.

## Why this is Mealy rather than Moore

The complete input starts a suffix word whose letters are state-indexed values, not the complete symbol itself.

## Edge cases fixed by the 7.x source

- Suffix is not emitted if the source errors.
- A scheduler argument is not a suffix value.

## Source anchors

- `src/internal/operators/endWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
