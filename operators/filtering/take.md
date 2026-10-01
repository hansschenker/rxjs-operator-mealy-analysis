# `take` — Mealy 6-tuple

| | |
|---|---|
| Category | Filtering |
| Kind | Pipeable operator |
| RxJS 7.x source | `src/internal/operators/take.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `take(count: number): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`take` is a pipeable operator on the RxJS 7.x line. Stable. Emit the first `count` source nexts and complete, unsubscribing from the source. `take(0)` completes immediately and does not subscribe to the source. A shorter source just completes.

In plain terms, the operator keeps this memory: S = { counting(k), stopped } for 0 ≤ k ≤ count. At subscription, before any source notification, that memory is counting(0). It reacts to these events: { subscribe, next(v), error, complete, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: take(2) on a b c writes next(a) next(b) complete and cancels before c.

Details that a marble diagram often leaves out: `count < 0` is treated as zero in 7.x and completes immediately. Unsubscribes on the completing next.

## Role in the notification machine

Emit the first `count` source nexts and complete, unsubscribing from the source. `take(0)` completes immediately and does not subscribe to the source. A shorter source just completes.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { counting(k), stopped }` for `0 ≤ k ≤ count`.

## 2. Initial state (S0)

`counting(0)`.

## 3. Input alphabet (Z)

`{ subscribe, next(v), error, complete, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(v), error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `counting(k) × next → counting(k+1)` if `k+1 < count`, else `stopped`.
- `subscribe` with count 0 → `stopped` without a source subscription.
- Source complete → `stopped`.

## 6. Output function (G : S × Z → A*)

- `next(v)` when `k+1 < count → next(v)`.
- `next(v)` when `k+1 = count → next(v) · complete`.
- `take(0)` subscribe → `complete`.
- Early source complete → `complete`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { counting(k), stopped } for 0 ≤ k ≤ count.

Memory at subscribe: counting(0).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| counting(k) × next | counting(k+1) if k+1 < count, else stopped | next(v) |
| subscribe with count 0 | stopped without a source subscription | complete |
| Source complete | stopped | next(v) · complete |
| Early source complete | named by the output row; memory change is in the transition rows above | complete |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`take(2)` on `a b c` writes `next(a) next(b) complete` and cancels before `c`.

## Why this is Mealy rather than Moore

The count-reaching next writes two letters, `next·complete`. Earlier nexts write one. The input is the same kind; state picks the word length.

## Edge cases fixed by the 7.x source

- `count < 0` is treated as zero in 7.x and completes immediately.
- Unsubscribes on the completing next.

## Source anchors

- `src/internal/operators/take.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
