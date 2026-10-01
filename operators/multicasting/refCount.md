# `refCount` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Connectable adapter |
| RxJS 7.x source | `src/internal/operators/refCount.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `refCount(): OperatorFunction<T, T>` |
| Status on the 7.x line | Deprecated. Use `share`. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

For a ConnectableObservable, the first subscriber calls `connect()`, further subscribers increment a count, and the last unsubscribe disconnects. Late subscribers do not replay unless the underlying subject does.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ refCount, connection | ⊥, stopped }`.

## 2. Initial state (S0)

refCount 0, no connection.

## 3. Input alphabet (Z)

`{ subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Join at 0 → connect, refCount 1.
- Join increments.
- Leave decrements; at 0 disconnect.
- Source terminal stops the subject.

## 6. Output function (G : S × Z → A*)

- Source next → next to current subscribers.
- Join writes ε unless the subject replays.

## Worked trace

Two subscribers share one connection. When both leave, the source is unsubscribed.

## Why this is Mealy rather than Moore

Join and leave are inputs that change connection without themselves being value letters. Source next writes a letter only while connected.

## Edge cases fixed by the 7.x source

- Deprecated in favor of `share`.
- Disconnect does not reset a sticky subject error by itself.

## Source anchors

- `src/internal/operators/refCount.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
