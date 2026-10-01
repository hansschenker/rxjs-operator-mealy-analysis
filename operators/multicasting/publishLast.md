# `publishLast` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable connectable operator |
| RxJS 7.x source | `src/internal/operators/publishLast.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `publishLast(): OperatorFunction<T, T>` |
| Status on the 7.x line | Deprecated. AsyncSubject-backed multicast. |

SuperGrok is the main contributor of this analysis.

## Explanation

`publishLast` is a pipeable connectable operator on the RxJS 7.x line. Deprecated. AsyncSubject-backed multicast. Subject is an `AsyncSubject`. It emits only the last source value, and only when the source completes, to current and late subscribers. Error is sticky.

In plain terms, the operator keeps this memory: S = { disconnected, connected(last | ⊥), stopped(last | error) }. At subscription, before any source notification, that memory is disconnected, no last. It reacts to these events: multicast alphabet.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Source 1, 2, complete after connect writes next(2) complete to subscribers. The 1 is overwritten.

Details that a marble diagram often leaves out: No value before complete. Deprecated.

## Role in the notification machine

Subject is an `AsyncSubject`. It emits only the last source value, and only when the source completes, to current and late subscribers. Error is sticky.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { disconnected, connected(last | ⊥), stopped(last | error) }`.

## 2. Initial state (S0)

`disconnected`, no last.

## 3. Input alphabet (Z)

`multicast` alphabet.

## 4. Output alphabet (A)

`{ next(last), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Source next overwrites last.
- Source complete → stopped and subject emits.
- Late join after stop still reads the AsyncSubject cache.

## 6. Output function (G : S × Z → A*)

- Source next → `ε` to subscribers.
- Source complete → `next(last) · complete` if a last exists, else `complete`.
- Late `subscriberJoin` after stop replays that word.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { disconnected, connected(last \| ⊥), stopped(last \| error) }.

Memory at subscribe: disconnected, no last.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Source next overwrites last | Source next overwrites last | ε to subscribers |
| Source complete | stopped and subject emits | next(last) · complete if a last exists, else complete |
| Late join after stop still reads the AsyncSubject cache | Late join after stop still reads the AsyncSubject cache | Late subscriberJoin after stop replays that word |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Source `1, 2, complete` after connect writes `next(2) complete` to subscribers. The `1` is overwritten.

## Why this is Mealy rather than Moore

Complete input writes the stored last. Next input writes `ε` and updates last. AsyncSubject replay is `G(stopped, subscriberJoin)`.

## Edge cases fixed by the 7.x source

- No value before complete.
- Deprecated.

## Source anchors

- `src/internal/operators/publishLast.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
