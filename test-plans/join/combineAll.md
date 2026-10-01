# Test plan: `combineAll`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join/combineAll.md](../../operators/join/combineAll.md).

| | |
|---|---|
| Category | Join |
| Kind | Deprecated alias |
| RxJS 7.x source | `src/internal/operators/combineAll.ts` |
| Signature | `combineAll(project?): OperatorFunction<ObservableInput<T>, T[]>` |
| Status | Deprecated alias of `combineLatestAll`. |

## Behavior under test

`combineAll` is a one-line re-export of `combineLatestAll`. The machine is the higher-order combineLatest machine: collect inners until the outer completes, then emit snapshots once every inner has a latest.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Same as `combineLatestAll`: collected inners, latest per inner, outerDone, stopped.

Memory at subscribe: No inners collected.

Events: `{ outerNext(inner$), outerComplete, outerError, innerNext_i, innerError_i, innerComplete_i, unsubscribe }`.

Possible notifications: `{ next(tuple), error, complete }`.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Identical to `combineLatestAll`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Deprecation does not add a state.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Identical to `combineLatestAll`: an inner next writes a snapshot only when every collected inner has a latest.`
5. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
6. Source edge 1: `File is a re-export. Behavior lives in `combineLatestAll.ts`.`
7. Source edge 2: `Removed in later majors.`

## Sequence from the analysis

Same word as `combineLatestAll` on the same higher-order source.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
