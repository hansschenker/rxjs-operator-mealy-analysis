# `switchMap` — Mealy 6-tuple

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/switchMap.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `switchMap(project, resultSelector?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. `resultSelector` deprecated. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Project each outer value to an inner and subscribe, unsubscribing any previous inner. Only the latest inner can emit. Complete when outer is done and the latest inner is done (or none is active).

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active inner | ⊥, outerDone, stopped }`.

## 2. Initial state (S0)

No inner, outer not done.

## 3. Input alphabet (Z)

Higher-order alphabet.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `outerNext` replaces `active` (unsubscribe previous).
- `innerNext` keeps `active`.
- `innerComplete` clears `active`; stop if outer done.
- Errors → `stopped`.

## 6. Output function (G : S × Z → A*)

- `innerNext → next`.
- `outerNext → ε`.
- Stale inner signals are not in Z after unsubscribe.
- `innerComplete → complete` if outer done, else `ε`.

## Worked trace

Search box: each keystroke switches the request inner. A slow response for an old key does not emit.

## Why this is Mealy rather than Moore

`outerNext` changes which inner's later inputs are legal. Output of an inner next is forwarded only because state still names that inner.

## Edge cases fixed by the 7.x source

- No queue. Previous inners are cancelled, not exhausted.
- Project throw errors immediately.

## Source anchors

- `src/internal/operators/switchMap.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
