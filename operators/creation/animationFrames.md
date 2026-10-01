# `animationFrames` — Mealy 6-tuple

| | |
|---|---|
| Category | Creation |
| Kind | Scheduled DOM producer |
| RxJS 7.x source | `src/internal/observable/dom/animationFrames.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `animationFrames(timestampProvider?): Observable<TimestampedAnimationFrame>` |
| Status on the 7.x line | Stable. |

SuperGrok is the main contributor of this analysis.

## Explanation

`animationFrames` is a scheduled DOM producer on the RxJS 7.x line. Stable. Subscribe schedules `requestAnimationFrame`. Each frame writes `{ timestamp, elapsed }` and schedules the next frame. Unsubscribe cancels the pending frame. No complete.

In plain terms, the operator keeps this memory: { idle, framing(startTime), stopped }. At subscription, before any source notification, that memory is idle. It reacts to these events: { subscribe, frame(timestamp), unsubscribe }.

A value is not automatically forwarded. What is sent depends on the memory and on the event that just arrived. Silence is a real result. One event may also send a value and then completion. After an error, a completion, or an unsubscribe, the operator is finished and later events are ignored.

Walk from the analysis: Three frames write three nexts whose elapsed values increase. Unsubscribe stops the loop.

Details that a marble diagram often leaves out: Browser-only in the DOM build. Each subscriber has a private frame loop.

## Role in the notification machine

Subscribe schedules `requestAnimationFrame`. Each frame writes `{ timestamp, elapsed }` and schedules the next frame. Unsubscribe cancels the pending frame. No complete.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`{ idle, framing(startTime), stopped }`.

## 2. Initial state (S0)

`idle`.

## 3. Input alphabet (Z)

`{ subscribe, frame(timestamp), unsubscribe }`.

## 4. Output alphabet (A)

`{ next({ timestamp, elapsed }) }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- Subscribe → framing(now).
- Frame stays framing and arms the next frame.
- Unsubscribe → stopped and cancelAnimationFrame.

## 6. Output function (G : S × Z → A*)

- Frame → next({ timestamp, elapsed: timestamp - startTime }).
- Subscribe and unsubscribe write ε.

## State transition table

Read a row as one step of the operator. The current memory and the event decide the next memory and what is sent. Nothing sent is a real result. One event may send more than one notification.

Memory: { idle, framing(startTime), stopped }.

Memory at subscribe: idle.

| Current memory and event | Next memory | Sent downstream |
|---|---|---|
| Subscribe | framing(now) | Subscribe and unsubscribe write ε |
| Frame stays framing | Frame stays framing and arms the next frame | next({ timestamp, elapsed: timestamp - startTime }) |
| Unsubscribe | stopped and cancelAnimationFrame | nothing named on a separate output row |
| finished, any later event | finished | nothing |

The finished row is absorbing: after an error, a completion, or an unsubscribe, a later event does not change memory and does not send a notification.

## Worked trace

Three frames write three nexts whose elapsed values increase. Unsubscribe stops the loop.

## Why this is Mealy rather than Moore

The frame input writes a letter computed from startTime in state and the timestamp on the input.

## Edge cases fixed by the 7.x source

- Browser-only in the DOM build.
- Each subscriber has a private frame loop.

## Source anchors

- `src/internal/observable/dom/animationFrames.ts`.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
