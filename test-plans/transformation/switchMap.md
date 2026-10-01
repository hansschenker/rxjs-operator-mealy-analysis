# Test plan: `switchMap`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/transformation/switchMap.md](../../operators/transformation/switchMap.md).

| | |
|---|---|
| Category | Transformation |
| Kind | Pipeable higher-order operator |
| RxJS 7.x source | `src/internal/operators/switchMap.ts` |
| Signature | `switchMap(project, resultSelector?): OperatorFunction<T, R>` |
| Status | Stable. `resultSelector` deprecated. |

## Behavior under test

Project each outer value to an inner and subscribe, unsubscribing any previous inner. Only the latest inner can emit. Complete when outer is done and the latest inner is done (or none is active).

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { active inner | ⊥, outerDone, stopped }`.

Memory at subscribe: No inner, outer not done.

Events: Higher-order alphabet.

Possible notifications: `{ next, error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerNext` replaces `active` (unsubscribe previous).`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerNext` keeps `active`.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``innerComplete` clears `active`; stop if outer done.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Errors → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerNext → next`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``outerNext → ε`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Stale inner signals are not in Z after unsubscribe.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. ``innerComplete → complete` if outer done, else `ε`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `No queue. Previous inners are cancelled, not exhausted.`
12. Source edge 2: `Project throw errors immediately.`

## Sequence from the analysis

Search box: each keystroke switches the request inner. A slow response for an old key does not emit.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
