# `share` — Mealy 6-tuple

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable refcounted multicast |
| RxJS 7.x source | `src/internal/operators/share.ts` |
| Tree revision used | `7.x @ e5351d02e225e275ac0e497c7b66eaa5f0c88791` |
| Signature (7.x) | `share(config?: ShareConfig<T>): MonoTypeOperatorFunction<T>` |
| Status on the 7.x line | Stable. Preferred multicast on 7.x. |

SuperGrok is the main contributor of this analysis.

## Role in the notification machine

Refcounted multicast. First subscriber connects via `connector()` (default `() => new Subject()`). Further subscribers join that subject. Defaults: `resetOnError: true`, `resetOnComplete: true`, `resetOnRefCountZero: true`, so the machine returns to cold when the source terminates or the last subscriber leaves. Each reset flag may be a boolean or a notifier factory.

An RxJS operator is modeled here as a Mealy machine because the word it writes downstream is a function of the **current memory** and the **notification that just arrived**, not of the state alone. The empty word is written `ε`. A stopped machine is absorbing: `T(stopped, z) = stopped` and `G(stopped, z) = ε`.

## 1. State space (S)

`S = { cold, hot(subject, refCount), resetting, stopped }`.

## 2. Initial state (S0)

`cold`.

## 3. Input alphabet (Z)

`{ subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete, resetTick, unsubscribe }`.

## 4. Output alphabet (A)

`{ next, error, complete }`.

`G` returns a finite word in `A*`. One input may therefore produce several notifications (`next · complete`) or none (`ε`).

## 5. Transition function (T : S × Z → S)

- `cold × subscriberJoin → hot(refCount=1)` and subscribe to source.
- Further joins increment refCount.
- Leave decrements. At 0, if resetOnRefCountZero, go `cold` after the optional notifier.
- Source error/complete reset to `cold` if the matching flag is true, else stay terminated on the same subject.

## 6. Output function (G : S × Z → A*)

- `sourceNext → next` to all current subscribers.
- `subscriberJoin` in `hot` writes whatever the connector subject replays (a plain Subject writes `ε`).
- Reset itself writes `ε` besides the terminal notification already sent.

## Worked trace

Two subscribers share one interval. Both unsubscribe: default share tears down and the next subscriber starts a fresh interval at 0.

## Why this is Mealy rather than Moore

Join, leave, and source terminal are different inputs that consult the reset flags in the configuration and the refCount state to decide the next state. Output of a source next is fan-out only while hot.

## Edge cases fixed by the 7.x source

- `resetOnError: false` makes a late subscriber receive the sticky error.
- `shareReplay` is share with a ReplaySubject connector and different reset defaults.
- Config notifiers delay the reset transition.

## Source anchors

- `src/internal/operators/share.ts`.
- 7.x `ShareConfig` flags default to true.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
