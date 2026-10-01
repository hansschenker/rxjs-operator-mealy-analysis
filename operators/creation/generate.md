# `generate` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold synchronous or scheduled generator |
| RxJS 7.x source | `src/internal/observable/generate.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `generate(initialState, condition, iterate, resultSelector?, scheduler?): Observable<T>` |
| Status on the 7.x line | Stable. Also accepts a `GenerateOptions` object. `resultSelector` form is the older overload. |

SuperGrok is the main contributor of this analysis.

## Explanation

`generate` is a cold synchronous or scheduled generator on the RxJS 7.x line. Stable. Also accepts a `GenerateOptions` object. `resultSelector` form is the older overload. `generate` is already a state loop. Subscription seeds `state`, then while `condition(state)` holds it emits `resultSelector(state)` and replaces state with `iterate(state)`. Completion is the condition failing. A scheduler turns each step into a scheduled input rather than a synchronous loop.

In plain terms, the operator keeps this memory: S = { idle, running(state), stopped }. state is the generator state, an arbitrary value, so S is infinite in general. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, step, unsubscribe }. Without a scheduler, step is the recursive continuation of subscribe.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: generate(1, s => s <= 3, s => s+1) writes next(1) next(2) next(3) complete. States visited: running(1), running(2), running(3), stopped.

Details that a marble diagram often leaves out: Options form (`initialState`, `condition`, `iterate`, `resultSelector`, `scheduler`) matches the tuple above. An infinite condition never writes `complete` unless unsubscribed.

## Role in the notification machine

`generate` is already a state loop. Subscription seeds `state`, then while `condition(state)` holds it emits `resultSelector(state)` and replaces state with `iterate(state)`. Completion is the condition failing. A scheduler turns each step into a scheduled input rather than a synchronous loop.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, running(state), stopped }`. `state` is the generator state, an arbitrary value, so `S` is infinite in general.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, step, unsubscribe }`. Without a scheduler, `step` is the recursive continuation of subscribe.

## 4. Output alphabet (A)

`{ next(result), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → running(initial)` if `condition(initial)` is true, else `stopped`.
- `running(s) × step → running(iterate(s))` if `condition(iterate(s))`, else `stopped`.
- A throw in condition, iterate, or result selector → `stopped`.
- `unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `idle × subscribe → next(result(initial))` if the condition holds, else `complete`.
- `running(s) × step → next(result(s'))` when the loop continues; `complete` when the condition fails after iterate.
- Throw → `error(e)`.

## Worked trace

`generate(1, s => s <= 3, s => s+1)` writes `next(1) next(2) next(3) complete`. States visited: `running(1)`, `running(2)`, `running(3)`, `stopped`.

## Why this is Mealy rather than Moore

This is the textbook case. `G(running(s), step)` projects `s` (or `iterate(s)`, depending on which moment the result is taken) while `T` advances `s`. The output letter is a function of both.

## Edge cases fixed by the 7.x source

- Options form (`initialState`, `condition`, `iterate`, `resultSelector`, `scheduler`) matches the tuple above.
- An infinite condition never writes `complete` unless unsubscribed.

## Source anchors

- `src/internal/observable/generate.ts`.
- Conceptually the closest creation function to an explicit Mealy generator.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
