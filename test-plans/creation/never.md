# Test plan: `never`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/creation/never.md](../../operators/creation/never.md).

| | |
|---|---|
| Category | Creation |
| Kind | Cold constant that never terminates |
| RxJS 7.x source | `src/internal/observable/never.ts` |
| Signature | `never(): Observable<never>  /  NEVER` |
| Status | Stable. `NEVER` is the constant; `never()` is the creation function. |

## Behavior under test

Subscribe and then emit nothing, never complete, never error. Only unsubscribe leaves the machine.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `{ idle, subscribed, stopped }`.

Memory at subscribe: `idle`.

Events: `{ subscribe, unsubscribe }`.

Possible notifications: Empty. No notification letters.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``idle × subscribe → subscribed`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``subscribed × unsubscribe → stopped`.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Every input writes `ε`.`
5. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
6. Source edge 1: `Useful as a default notifier that never fires.`
7. Source edge 2: `Does not schedule anything.`

## Sequence from the analysis

A subscriber to `NEVER` receives no next, no error, and no complete until it unsubscribes.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
