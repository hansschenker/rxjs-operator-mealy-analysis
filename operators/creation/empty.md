# `empty` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function / constant |
| RxJS 7.x source | `src/internal/observable/empty.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `empty(scheduler?: SchedulerLike): Observable<never>` |
| Status on the 7.x line | Deprecated on 7.x in favor of the `EMPTY` constant. Still present. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

The machine emits no values. Subscribe writes `complete` and stops. An optional scheduler only delays that single complete notification; it does not add values.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, scheduled, stopped }`. Without a scheduler, `scheduled` is skipped.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, schedulerTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → stopped` if no scheduler.
- `idle × subscribe → scheduled` if a scheduler is given.
- `scheduled × schedulerTick → stopped`.
- `scheduled × unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `idle × subscribe → complete` (synchronous path).
- `scheduled × schedulerTick → complete`.
- Otherwise `ε`.

## Worked trace

`empty().subscribe(observer)` produces the word `complete` and nothing else.

## Why this is Mealy rather than Moore

Trivial Mealy: the only productive input is subscribe (or the scheduled tick that subscribe armed). Output is determined by that input, not by sitting in a state.

## Edge cases fixed by the 7.x source

- `EMPTY` is the same machine with no scheduler argument.
- Deprecated status does not change the tuple.

## Source anchors

- `src/internal/observable/empty.ts` exports the creation function; `EMPTY` is the constant.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
