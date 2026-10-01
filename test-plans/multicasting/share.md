# Test plan: `share`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/multicasting/share.md](../../operators/multicasting/share.md).

| | |
|---|---|
| Category | Multicasting |
| Kind | Pipeable refcounted multicast |
| RxJS 7.x source | `src/internal/operators/share.ts` |
| Signature | `share(config?: ShareConfig<T>): MonoTypeOperatorFunction<T>` |
| Status | Stable. Preferred multicast on 7.x. |

## Behavior under test

Refcounted multicast. First subscriber connects via `connector()` (default `() => new Subject()`). Further subscribers join that subject. Defaults: `resetOnError: true`, `resetOnComplete: true`, `resetOnRefCountZero: true`, so the machine returns to cold when the source terminates or the last subscriber leaves. Each reset flag may be a boolean or a notifier factory.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { cold, hot(subject, refCount), resetting, stopped }`.

Memory at subscribe: `cold`.

Events: `{ subscriberJoin, subscriberLeave, sourceNext, sourceError, sourceComplete, resetTick, unsubscribe }`.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``cold × subscriberJoin → hot(refCount=1)` and subscribe to source.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Further joins increment refCount.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Leave decrements. At 0, if resetOnRefCountZero, go `cold` after the optional notifier.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Source error/complete reset to `cold` if the matching flag is true, else stay terminated on the same subject.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``sourceNext → next` to all current subscribers.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``subscriberJoin` in `hot` writes whatever the connector subject replays (a plain Subject writes `ε`).`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Reset itself writes `ε` besides the terminal notification already sent.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: ``resetOnError: false` makes a late subscriber receive the sticky error.`
11. Source edge 2: ``shareReplay` is share with a ReplaySubject connector and different reset defaults.`
12. Source edge 3: `Config notifiers delay the reset transition.`

## Sequence from the analysis

Two subscribers share one interval. Both unsubscribe: default share tears down and the next subscriber starts a fresh interval at 0.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
