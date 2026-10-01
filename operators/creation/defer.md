# `defer` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold factory |
| RxJS 7.x source | `src/internal/observable/defer.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `defer(observableFactory: () => ObservableInput<T>): Observable<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`defer` is a cold factory on the RxJS 7.x line. Stable. `defer` has almost no memory of its own. Subscription evaluates the factory and subscribes to whatever `ObservableInput` comes back. Subsequent notifications are forwarded verbatim. A factory throw is an error on that subscription only.

In plain terms, the operator keeps this memory: S = { idle, forwarding, stopped }. forwarding means the inner subscription is live. The factory result is not stored as a value cache. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, innerNext(v), innerError(e), innerComplete, unsubscribe }. Factory failure is folded into subscribe.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Two subscribers call the factory twice. Each machine starts at idle. There is no shared inner.

Details that a marble diagram often leaves out: The factory runs per subscription. That is the whole point versus `of(factory())` evaluated early. Returned promises, iterables, and arrays are normalized by `from` semantics inside subscribe.

## Role in the notification machine

`defer` has almost no memory of its own. Subscription evaluates the factory and subscribes to whatever `ObservableInput` comes back. Subsequent notifications are forwarded verbatim. A factory throw is an error on that subscription only.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { idle, forwarding, stopped }`. `forwarding` means the inner subscription is live. The factory result is not stored as a value cache.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, innerNext(v), innerError(e), innerComplete, unsubscribe }`. Factory failure is folded into `subscribe`.

## 4. Output alphabet (A)

`{ next(v), error(e), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → forwarding` if the factory returns an input.
- `idle × subscribe → stopped` if the factory throws.
- `forwarding × innerNext → forwarding`.
- `forwarding × innerError | innerComplete | unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `idle × subscribe → ε` on success; `error(e)` if the factory throws.
- `forwarding × innerNext(v) → next(v)`.
- `forwarding × innerError(e) → error(e)`.
- `forwarding × innerComplete → complete`.
- `unsubscribe → ε`.

## Worked trace

Two subscribers call the factory twice. Each machine starts at `idle`. There is no shared inner.

## Why this is Mealy rather than Moore

`subscribe` writes either `ε` or `error` depending on the factory outcome, which is part of the input event. Forwarded notifications are the identity Mealy map.

## Edge cases fixed by the 7.x source

- The factory runs per subscription. That is the whole point versus `of(factory())` evaluated early.
- Returned promises, iterables, and arrays are normalized by `from` semantics inside subscribe.

## Source anchors

- `src/internal/observable/defer.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
