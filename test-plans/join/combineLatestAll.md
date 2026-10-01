# Test plan: `combineLatestAll`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/combineLatestAll.md](../../operators/join/combineLatestAll.md).

| | |
|---|---|
| Category | Join |
| Kind | Pipeable higher-order join |
| RxJS 7.x source | `src/internal/operators/combineLatestAll.ts` |
| Signature | `combineLatestAll(project?): OperatorFunction<ObservableInput<T>, T[]>` |
| Status | Stable. Successor of deprecated `combineAll`. |

## Behavior under test

Collect the inners emitted by the source. When the source completes, combineLatest those inners: emit when each has a latest, then on any inner next. If the source completes with no inners, complete without a next.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: `S = { collected inners, outerDone, latest per inner, stopped }`.

Memory at subscribe: No inners.

Events: `{ outerNext(inner$), outerComplete, outerError, innerNext_i, innerError_i, innerComplete_i, unsubscribe }`.

Possible notifications: `{ next(tuple), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerNext` appends an inner and subscribes.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. ``outerComplete` freezes the set.`
4. Transition 3: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Inner next stores latest.`
5. Transition 4: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `All inners complete after ready → `stopped`.`
6. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Before every collected inner has a value → `ε`.`
7. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Inner next once ready → `next(snapshot)`.`
8. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `All complete → `complete`.`
9. Output 4: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Any error → `error`.`
10. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
11. Source edge 1: `Inners that arrive after outer complete are not expected; outer complete closes collection.`
12. Source edge 2: `Empty higher-order source completes.`

## Sequence from the analysis

Source emits two inners then completes. First combined next appears only after both inners have emitted.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
