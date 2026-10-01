# `distinct` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/distinct.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `distinct(keySelector?, flushes?): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`distinct` is a pipeable operator on the RxJS 7.x line. Stable. Emit a value only the first time its key is seen. Key defaults to the value itself. Memory is a set. Optional `flushes` observable clears the set when it emits.

In plain terms, the operator keeps this memory: S = { seen: Set, stopped }. At subscription, before any source notification, that memory is Empty set. It reacts to these events: { next(v), error, complete, flush, flushError, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 1, 2, 1).pipe(distinct()) writes next(1) next(2) complete.

Details that a marble diagram often leaves out: Set membership uses the key selector result. Unbounded memory if the key domain is unbounded and no flush is given.

## Role in the notification machine

Emit a value only the first time its key is seen. Key defaults to the value itself. Memory is a set. Optional `flushes` observable clears the set when it emits.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { seen: Set, stopped }`.

## 2. Initial state (S0)

Empty set.

## 3. Input alphabet (Z)

`{ next(v), error, complete, flush, flushError, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next(v)` adds `key(v)` if absent.
- `flush` clears the set.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v) → next(v)` if key was absent, else `ε`.
- `flush → ε`.
- Error and complete copy through.

## Worked trace

`of(1, 1, 2, 1).pipe(distinct())` writes `next(1) next(2) complete`.

## Why this is Mealy rather than Moore

Emission is a function of the set state and the arriving key. Flush changes later outputs without itself emitting.

## Edge cases fixed by the 7.x source

- Set membership uses the key selector result.
- Unbounded memory if the key domain is unbounded and no flush is given.

## Source anchors

- `src/internal/operators/distinct.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
