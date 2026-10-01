# `concat` — Mealy 6-tuple

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/concat.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `concat(...sources): Observable<T>` |
| Status on the 7.x line | Stable. Pipeable cousin is `concatWith` / `concatAll`. |

SuperGrok is the main contributor of this analysis.

## Explanation

`concat` is a join creation function on the RxJS 7.x line. Stable. Pipeable cousin is `concatWith` / `concatAll`. Subscribe to the first source and forward it until it completes, then subscribe to the next, and so on. One active inner. An error from any source errors the output and later sources are not subscribed. Complete after the last source completes.

In plain terms, the operator keeps this memory: S = { reading(i) | 0 ≤ i ≤ n } ∪ { stopped }. reading(i) means source i is the active subscription. At subscription, before any source notification, that memory is reading(0) after subscribe; n = 0 (no sources) is already done. It reacts to these events: { subscribe, innerNext(v), innerError(e), innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: concat(of(1,2), of(3)) writes next(1) next(2) next(3) complete. The 3 cannot appear before the first source completes.

Details that a marble diagram often leaves out: Sources are subscribed lazily, not up front. Promises and arrays are normalized via `from` when they are reached.

## Role in the notification machine

Subscribe to the first source and forward it until it completes, then subscribe to the next, and so on. One active inner. An error from any source errors the output and later sources are not subscribed. Complete after the last source completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { reading(i) | 0 ≤ i ≤ n } ∪ { stopped }`. `reading(i)` means source `i` is the active subscription.

## 2. Initial state (S0)

`reading(0)` after subscribe; `n = 0` (no sources) is already done.

## 3. Input alphabet (Z)

`{ subscribe, innerNext(v), innerError(e), innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `reading(i) × innerNext → reading(i)`.
- `reading(i) × innerComplete → reading(i+1)` if `i+1 < n`, else `stopped`.
- `reading(i) × innerError → stopped`.
- `unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext(v) → next(v)`.
- `innerError(e) → error(e)`.
- `innerComplete → ε` if another source remains; `complete` if it was the last.

## Worked trace

`concat(of(1,2), of(3))` writes `next(1) next(2) next(3) complete`. The `3` cannot appear before the first source completes.

## Why this is Mealy rather than Moore

`innerComplete` writes either `ε` (and a new subscription action) or `complete`, depending on the index stored in state.

## Edge cases fixed by the 7.x source

- Sources are subscribed lazily, not up front.
- Promises and arrays are normalized via `from` when they are reached.

## Source anchors

- `src/internal/observable/concat.ts`, implemented as `concatAll` over `from(sources)`.
- See also `src/internal/operators/concatAll.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
