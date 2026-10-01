# `ignoreElements` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/ignoreElements.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `ignoreElements(): OperatorFunction<T, never>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`ignoreElements` is a pipeable operator on the RxJS 7.x line. Stable. Drop every next. Forward error and complete only.

In plain terms, the operator keeps this memory: S = { active, stopped }. At subscription, before any source notification, that memory is active. It reacts to these events: { next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: of(1, 2, 3).pipe(ignoreElements()) writes complete.

Details that a marble diagram often leaves out: Useful to keep a stream's terminal signals while discarding values. Does not delay complete.

## Role in the notification machine

Drop every next. Forward error and complete only.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { active, stopped }`.

## 2. Initial state (S0)

`active`.

## 3. Input alphabet (Z)

`{ next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `next` stays `active`.
- Terminal → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next → ε`.
- `error → error`, `complete → complete`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { active, stopped }.

Memory at subscribe: active.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| next stays active | next stays active | ε |
| Terminal | stopped | error, complete → complete |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`of(1, 2, 3).pipe(ignoreElements())` writes `complete`.

## Why this is Mealy rather than Moore

Next and complete are different inputs in the same state and write different words.

## Edge cases fixed by the 7.x source

- Useful to keep a stream's terminal signals while discarding values.
- Does not delay complete.

## Source anchors

- `src/internal/operators/ignoreElements.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
