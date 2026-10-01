# Test plan: `partition`

SuperGrok is the main contributor of this test plan.

Derived from the 6-tuple analysis [operators/join-creation/partition.md](../../operators/join-creation/partition.md).

| | |
|---|---|
| Category | Join creation |
| Kind | Splitting function, not a pipeable operator |
| RxJS 7.x source | `src/internal/operators/partition.ts` |
| Signature | `partition(source, predicate, thisArg?): [Observable<T>, Observable<T>]` |
| Status | Stable function. Listed by the docs under both join creation and transformation. Not used inside `pipe`. |

## Behavior under test

`partition` returns two observables: values for which `predicate` is true, and values for which it is false. On 7.x it is implemented as two `filter` subscriptions, not as one shared multicast. A cold source therefore runs twice, once per branch, unless the caller `share`s it first. The Mealy model below is the logical splitter; the source note records the double subscription.

## What a case asserts

Each case fixes the memory, delivers one event, then checks two results: the memory afterwards, and the notifications sent. Sending nothing is a result. Sending a value and then completion is one result. Unsubscribe, abort, and inner subscribe are assertions when the analysis names them as actions.

## Memory and events

Memory: Logical splitter: `S = { active(i), stopped }` with `i` the source index passed to the predicate. Implementation: two independent filter machines.

Memory at subscribe: `active(0)` per branch subscription.

Events: `{ next(v), error(e), complete, unsubscribe }` on each subscription.

Possible notifications: Branch A: `{ next(v) if predicate, error, complete }`. Branch B: the negation.

## Cases

1. Subscribe, then assert the initial memory before any source notification. No notification is expected unless the analysis says subscribe itself writes one.
2. Transition 1: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Each branch: `active(i) × next(v) → active(i+1)`.`
3. Transition 2: arrange the memory named in this row, deliver the named event, and assert the resulting memory. `Terminal inputs → `stopped` on that branch only.`
4. Output 1: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Pass branch: `next(v) → next(v)` if `predicate(v, i)`, else `ε`.`
5. Output 2: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Fail branch: `next(v) → next(v)` if not predicate, else `ε`.`
6. Output 3: arrange the same memory, deliver the same event, and assert the downstream word, including silence. `Error and complete are copied to the branch that received them.`
7. After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays stopped and nothing is sent.
8. Source edge 1: `Not a single subscription. Side-effecting sources run per branch.`
9. Source edge 2: `Index is the index in that branch's subscription, which coincide only if both are subscribed and the source is deterministic.`
10. Source edge 3: ``thisArg` is deprecated style.`

## Sequence from the analysis

`partition(of(1,2,3), x => x % 2)` yields pass word `next(1) next(3) complete` and fail word `next(2) complete`, but `of` is subscribed twice.

## How to encode it

Use a RxJS `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the scheduler's subscription marble when the analysis says the source is cancelled. Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
