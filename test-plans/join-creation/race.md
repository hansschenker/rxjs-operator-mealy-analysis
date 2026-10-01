# Test plan: `race`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join-creation/race.md](../../operators/join-creation/race.md).

| | |
|---|---|
| Category | Join creation |
| Kind | Join creation function |
| RxJS 7.x source | `src/internal/observable/race.ts` |
| Signature | `race(...sources): Observable<T>` |
| Status | Stable. Pipeable cousin is `raceWith`. |

## Behavior under test

Subscribe to all sources. The first source to emit a next (or to terminate, in the 7.x race implementation the first notifier wins) becomes the winner; the others are unsubscribed. Forward the winner until it terminates.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { racing, forwarding(winner), stopped }`.

Memory at subscribe: `racing` after subscribe.

Events: `{ subscribe, next_i, error_i, complete_i, unsubscribe }`.

Possible notifications: `{ next(v), error(e), complete }` plus the action of unsubscribing losers.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``racing × next_i → forwarding(i)` and drop other subscriptions.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``racing × error_i → stopped` if that error is the first signal (7.x: first notification wins, including error/complete).`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``racing × complete_i → stopped` if complete wins the race.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``forwarding(i)` copies i's transitions; signals from losers are not in Z anymore.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``racing × next_i(v) → next(v)`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``racing × error_i(e) → error(e)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``racing × complete_i → complete`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Later winner notifications are forwarded.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Empty race completes.`
12. Source edge 2: `Synchronous first source wins before later sources are fully armed only according to subscription order; a sync source earlier in the list wins.`

## Sequence from the analysis

`race(slow$, fastOf(1))` writes `next(1) complete` and unsubscribes `slow$` as soon as `1` arrives.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
