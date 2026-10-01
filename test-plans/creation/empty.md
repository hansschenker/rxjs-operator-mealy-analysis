# Test plan: `empty`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/empty.md](../../operators/creation/empty.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold creation function / constant |
| RxJS 7.x source | `src/internal/observable/empty.ts` |
| Signature | `empty(scheduler?: SchedulerLike): Observable<never>` |
| Status | Deprecated on 7.x in favor of the `EMPTY` constant. Still present. |

## Behavior under test

The machine emits no values. Subscribe writes `complete` and stops. An optional scheduler only delays that single complete notification; it does not add values.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { idle, scheduled, stopped }`. Without a scheduler, `scheduled` is skipped.

Memory at subscribe: `idle`.

Events: `{ subscribe, schedulerTick, unsubscribe }`.

Possible notifications: `{ complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → stopped` if no scheduler.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → scheduled` if a scheduler is given.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``scheduled × schedulerTick → stopped`.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``scheduled × unsubscribe → stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``idle × subscribe → complete` (synchronous path).`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``scheduled × schedulerTick → complete`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Otherwise `ε`.`
9. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
10. Source edge 1: ``EMPTY` is the same machine with no scheduler argument.`
11. Source edge 2: `Deprecated status does not change the tuple.`

## Sequence from the analysis

`empty().subscribe(observer)` produces the word `complete` and nothing else.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
