# `onErrorResumeNext` — Mealy 6-tuple

| | |
|---|---|
| Category | Error handling |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/onErrorResumeNext.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `onErrorResumeNext(...sources): Observable<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Creation form of `onErrorResumeNextWith`. Subscribe to the first source. On its error or complete, subscribe to the next. Swallow errors. Complete after the last source completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ reading(i), stopped }`.

## 2. Initial state (S0)

`reading(0)`.

## 3. Input alphabet (Z)

`{ innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Error or complete advances i.
- Last source terminal → stopped.

## 6. Output function (G : S × Z → A*)

- Inner next → next.
- Inner error → ε and continue, or complete if it was the last.
- Final complete → complete.

## Worked trace

`onErrorResumeNext(throwError(() => 'x'), of(1))` writes `next(1) complete`.

## Why this is Mealy rather than Moore

Error is an input that writes ε rather than error, then T moves to the next source.

## Edge cases fixed by the 7.x source

- Same continuation rule as the pipeable `onErrorResumeNextWith`.
- A source that completes also continues.

## Source anchors

- `src/internal/observable/onErrorResumeNext.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
