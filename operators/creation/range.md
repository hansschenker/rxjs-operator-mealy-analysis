# `range` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold numeric producer |
| RxJS 7.x source | `src/internal/observable/range.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `range(start: number, count?: number, scheduler?: SchedulerLike): Observable<number>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`range` is a cold numeric producer on the RxJS 7.x line. Stable. Subscribe emits `count` integers beginning at `start`, then completes. `count` defaults to `undefined` in older signatures but the 7.x form is `range(start, count?)`. Zero or negative count completes without values.

In plain terms, the operator keeps this memory: S = { pending(k) | 0 ≤ k ≤ count } ∪ { stopped }. At subscription, before any source notification, that memory is pending(0). It reacts to these events: { subscribe, step, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: range(2, 3) writes next(2) next(3) next(4) complete.

Details that a marble diagram often leaves out: `count <= 0` writes `complete` only. Scheduler spreads the word; it does not change it.

## Role in the notification machine

Subscribe emits `count` integers beginning at `start`, then completes. `count` defaults to `undefined` in older signatures but the 7.x form is `range(start, count?)`. Zero or negative count completes without values.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { pending(k) | 0 ≤ k ≤ count } ∪ { stopped }`.

## 2. Initial state (S0)

`pending(0)`.

## 3. Input alphabet (Z)

`{ subscribe, step, unsubscribe }`.

## 4. Output alphabet (A)

`{ next(start+k), complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `pending(k) × step → pending(k+1)` while `k < count`.
- `pending(count) × step → stopped`.
- `unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- `pending(k) × step → next(start+k)` for `k < count`.
- `pending(count) × step → complete`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: S = { pending(k) \| 0 ≤ k ≤ count } ∪ { stopped }.

Memory at subscribe: pending(0).

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| pending(k) × step | pending(k+1) while k < count | next(start+k) for k < count |
| pending(count) × step | stopped | complete |
| unsubscribe | stopped | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

`range(2, 3)` writes `next(2) next(3) next(4) complete`.

## Why this is Mealy rather than Moore

Same shape as `of`, with the letter computed from `(start, k)` rather than read from an argument list.

## Edge cases fixed by the 7.x source

- `count <= 0` writes `complete` only.
- Scheduler spreads the word; it does not change it.

## Source anchors

- `src/internal/observable/range.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
