# `connect` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable selector multicast |
| RxJS 7.x source | `src/internal/operators/connect.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `connect(selector, config?): OperatorFunction<T, R>` |
| Status on the 7.x line | Stable. The supported replacement for `multicast` with a selector. |

SuperGrok is the main contributor of this analysis.

## Explanation

`connect` is a pipeable selector multicast on the RxJS 7.x line. Stable. The supported replacement for `multicast` with a selector. On subscribe, build a connector subject (default Subject), subscribe the selector's result, and connect the source into the subject for the lifetime of that subscription. `config.connector` picks the subject. There is no manual `connect()` call.

In plain terms, the operator keeps this memory: { connecting(subject), stopped }. At subscription, before any source notification, that memory is Entered on subscribe by constructing the subject and calling the selector. It reacts to these events: { subscribe, sourceNext, sourceError, sourceComplete, selectorNext, selectorError, selectorComplete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: source.pipe(connect(shared => shared.pipe(take(2)))) shares one source subscription for the selector and completes when the selector completes, tearing down the connection.

Details that a marble diagram often leaves out: One connection per subscriber to the result, unless the selector itself shares. Connector factory runs per subscribe.

## Role in the notification machine

On subscribe, build a connector subject (default Subject), subscribe the selector's result, and connect the source into the subject for the lifetime of that subscription. `config.connector` picks the subject. There is no manual `connect()` call.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ connecting(subject), stopped }`.

## 2. Initial state (S0)

Entered on subscribe by constructing the subject and calling the selector.

## 3. Input alphabet (Z)

`{ subscribe, sourceNext, sourceError, sourceComplete, selectorNext, selectorError, selectorComplete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }` from the selector observable.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Subscribe connects source to subject and subscribes to `selector(subject)`.
- Source terminal completes or errors the subject.
- Selector terminal → stopped.
- Unsubscribe disconnects.

## 6. Output function (G : S × Z → A*)

- Selector notifications are the output word.
- Source next is delivered to the subject, which the selector may or may not forward.

## Worked trace

`source.pipe(connect(shared => shared.pipe(take(2))))` shares one source subscription for the selector and completes when the selector completes, tearing down the connection.

## Why this is Mealy rather than Moore

Subscribe is the input that arms the connection. Later output letters come from the selector machine, not automatically from every source next.

## Edge cases fixed by the 7.x source

- One connection per subscriber to the result, unless the selector itself shares.
- Connector factory runs per subscribe.

## Source anchors

- `src/internal/operators/connect.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
