# `onErrorResumeNextWith` — Mealy 6-tuple

| | |
|---|---|
| Category | Error handling |
| Kind | Pipeable continuation |
| RxJS 7.x source | `src/internal/operators/onErrorResumeNextWith.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `onErrorResumeNextWith(...nextSources): OperatorFunction<T, T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`onErrorResumeNextWith` is a pipeable continuation on the RxJS 7.x line. Stable. Forward the source. On error or complete, subscribe to the next continuation instead of failing. Every source is continued past its error. The output completes when the last continuation completes. An error is swallowed and becomes the signal to move on.

In plain terms, the operator keeps this memory: { reading(i), stopped }. At subscription, before any source notification, that memory is reading(0) on the piped source. It reacts to these events: { innerNext, innerError, innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A source that errors, continued with of(1), writes next(1) complete and no error.

Details that a marble diagram often leaves out: Both error and complete move to the next source. Creation cousin is `onErrorResumeNext`.

## Role in the notification machine

Forward the source. On error or complete, subscribe to the next continuation instead of failing. Every source is continued past its error. The output completes when the last continuation completes. An error is swallowed and becomes the signal to move on.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ reading(i), stopped }`.

## 2. Initial state (S0)

`reading(0)` on the piped source.

## 3. Input alphabet (Z)

`{ innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, complete }`. Errors are not in the normal output word.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `innerNext` stays.
- `innerError` or `innerComplete` advances i, or stops if i was last.
- Unsubscribe → stopped.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `innerError → ε` (switch action), unless it was the last source, in which case the sequence completes.
- `innerComplete → ε` or `complete` if nothing remains.

## Worked trace

A source that errors, continued with `of(1)`, writes `next(1) complete` and no error.

## Why this is Mealy rather than Moore

Error input writes ε and a resubscribe, not error. That input-dependent suppression is the operator.

## Edge cases fixed by the 7.x source

- Both error and complete move to the next source.
- Creation cousin is `onErrorResumeNext`.

## Source anchors

- `src/internal/operators/onErrorResumeNextWith.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
