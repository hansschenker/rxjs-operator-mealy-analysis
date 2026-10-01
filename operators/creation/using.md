# `using` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Resource factory |
| RxJS 7.x source | `src/internal/observable/using.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `using(resourceFactory, observableFactory): Observable<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Subscribe calls `resourceFactory()`, then `observableFactory(resource)`, and subscribes to that result. Unsubscribe or terminal disposes the resource. A factory throw errors the subscription and still disposes if the resource was created.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ idle, forwarding(resource), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, innerNext, innerError, innerComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Subscribe → forwarding if both factories return.
- Inner terminal or unsubscribe → stopped and dispose.
- Factory throw → stopped.

## 6. Output function (G : S × Z → A*)

- Inner notifications copy through.
- Factory throw → error.
- Dispose is an action on the ending input, not a notification.

## Worked trace

A resource opened on subscribe is closed when the inner completes or the consumer unsubscribes, even if no value was emitted.

## Why this is Mealy rather than Moore

Subscribe either writes ε and opens a resource or writes error. The ending input writes the terminal letter and the dispose action.

## Edge cases fixed by the 7.x source

- Resource factory runs per subscription.
- Disposal is tied to the subscription, not to garbage collection.

## Source anchors

- `src/internal/observable/using.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
