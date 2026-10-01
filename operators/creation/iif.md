# `iif` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold conditional factory |
| RxJS 7.x source | `src/internal/observable/iif.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `iif(condition: () => boolean, trueResult?: ObservableInput<T>, falseResult?: ObservableInput<F>): Observable<T|F>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`iif` is a cold conditional factory on the RxJS 7.x line. Stable. Subscribe evaluates `condition()` and subscribes to either the true or the false observable. Missing branch is `EMPTY` (immediate complete). After the choice, the machine is a forwarder. The condition is not re-checked.

In plain terms, the operator keeps this memory: S = { idle, forwarding(branch), stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, innerNext, innerError, innerComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: iif(() => flag, of(1), of(2)) with flag = true writes next(1) complete and never subscribes to of(2).

Details that a marble diagram often leaves out: Condition runs per subscription. Default branch completes empty.

## Role in the notification machine

Subscribe evaluates `condition()` and subscribes to either the true or the false observable. Missing branch is `EMPTY` (immediate complete). After the choice, the machine is a forwarder. The condition is not re-checked.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, forwarding(branch), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → forwarding(true)` or `forwarding(false)` based on `condition()`.
- A throw in `condition` → `stopped`.
- Inner terminal or unsubscribe → `stopped`.
- Inner next stays in `forwarding`.

## 6. Output function (G : S × Z → A*)

- `subscribe` throw → `error(e)`, else `ε` (branch subscription is an action).
- Inner notifications are copied to the output.
- Missing branch writes `complete` on subscribe.

## Worked trace

`iif(() => flag, of(1), of(2))` with `flag = true` writes `next(1) complete` and never subscribes to `of(2)`.

## Why this is Mealy rather than Moore

The subscribe input plus the condition result selects the branch; later outputs are the identity map on inner inputs. Choice is an input-time decision, not a Moore output of `idle`.

## Edge cases fixed by the 7.x source

- Condition runs per subscription.
- Default branch completes empty.

## Source anchors

- `src/internal/observable/iif.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
