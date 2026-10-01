# `never` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Cold constant that never terminates |
| RxJS 7.x source | `src/internal/observable/never.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `never(): Observable<never>  /  NEVER` |
| Status on the 7.x line | Stable. `NEVER` is the constant; `never()` is the creation function. |

SuperGrok is the main contributor of this analysis.

## Explanation

`never` is a cold constant that never terminates on the RxJS 7.x line. Stable. `NEVER` is the constant; `never()` is the creation function. Subscribe and then emit nothing, never complete, never error. Only unsubscribe leaves the machine.

In plain terms, the operator keeps this memory: { idle, subscribed, stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: A subscriber to NEVER receives no next, no error, and no complete until it unsubscribes.

Details that a marble diagram often leaves out: Useful as a default notifier that never fires. Does not schedule anything.

## Role in the notification machine

Subscribe and then emit nothing, never complete, never error. Only unsubscribe leaves the machine.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ idle, subscribed, stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, unsubscribe }`.

## 4. Output alphabet (A)

Empty. No notification letters.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `idle × subscribe → subscribed`.
- `subscribed × unsubscribe → stopped`.

## 6. Output function (G : S × Z → A*)

- Every input writes `ε`.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { idle, subscribed, stopped }.

Memory at subscribe: idle.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| idle × subscribe | subscribed | nothing named on a separate output row |
| subscribed × unsubscribe | stopped | nothing named on a separate output row |
| Every input writes ε | named by the output row; memory change is in the transition rows above | Every input writes ε |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

A subscriber to `NEVER` receives no next, no error, and no complete until it unsubscribes.

## Why this is Mealy rather than Moore

The machine is Mealy in the trivial sense: the only inputs write the empty word. It is the identity element for races that wait forever.

## Edge cases fixed by the 7.x source

- Useful as a default notifier that never fires.
- Does not schedule anything.

## Source anchors

- `src/internal/observable/never.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
