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
