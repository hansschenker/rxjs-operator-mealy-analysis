# Custom operator checklist

SuperGrok is the main contributor of this project.

Use this before writing an operator. Answer the six questions from [docs/methodology.md](methodology.md) first, then implement only those rows. A custom operator is the same kind of machine as the operators under `operators/`: it has memory, it reads one event at a time, and from the memory plus the event it decides the next memory and what to send.

Copy [docs/custom-operator-template.md](custom-operator-template.md), fill it in, and keep it next to the implementation. The filled checklist is the specification. The tests from [docs/test-plan.md](test-plan.md) are the rows of that specification.

## When to write the checklist

Write it when the operator is not already one of the analyzed RxJS operators, or when it is a deliberate variant of one. If the behavior is `switchMap`, `audit`, or `take`, use that analysis instead of inventing a near-copy. The checklist earns its place when the policy is yours: a queue limit, a custom flush rule, a resource that must be disposed, a second source that is not a standard notifier.

## The six questions, in order

Answer them in this order. Do not start from the marble diagram.

1. **What can it remember?** Name the memory in words. A counter, the latest value, an open buffer, the active inner, a queue, a subscriber count, a resource handle. Settings such as the projection, the duration, and the predicate are constants of this machine, not memory and not events. Also name the finished memory. Once the operator has errored, completed, or been unsubscribed, it stays there.

2. **What does it remember at subscribe, before any source notification?** This is the initial memory. If subscribe itself opens a request, starts a timer, or emits a prefix, say so here. Those are events or output of subscribe, not memory that existed before subscribe.

3. **What events can arrive?** Upstream value, upstream error, upstream completion, and unsubscribe are the default set. Add a timer tick if it schedules. Add inner value, inner error, and inner completion if it subscribes to something else. Tag events with their source if more than one source exists. Do not list events the operator never receives.

4. **What can it send?** Value, error, and completion. Silence is a result: the event was consumed and nothing was forwarded. One event may send a value and then completion. Name actions that are not notifications when they matter: unsubscribe an inner, abort a request, dispose a resource, call a teardown.

5. **For each memory and each event, what is remembered next?** One row per pair that can happen. A value while idle and a value while busy are different rows if they do different things. The finished memory stays finished for every later event.

6. **For that same pair, what is sent?** Silence, one notification, or several. The answer may depend on both the memory and the event. This is the Mealy part. A Moore description would try to emit from the memory alone and would hide which event caused the notification.

If you cannot walk a short sequence from these answers, a memory or an event is missing. Fix the checklist before writing code.

## What to decide explicitly

These are the rows marble diagrams usually skip. Write them down even if the answer is "nothing happens."

- Does completion flush a held value, or drop it?
- Does an error flush a held value, or drop it?
- Does unsubscribe cancel a timer, an inner, or a resource?
- Is a late tick after unsubscribe delivered?
- Does a second value restart a duration, or only replace the stored value?
- Is an inner queued, switched, exhausted, or merged, and what is the bound?
- Does the operator subscribe to the source immediately, or only after subscribe has done something else?
- What happens if the projection, the notifier factory, or the resource factory throws?

## Then implement only the rows

A practical shape in RxJS 7 is `operate` plus `createOperatorSubscriber`. The variables inside `operate` are the memory. The subscriber callbacks are the events. `subscriber.next`, `subscriber.error`, and `subscriber.complete` are the output word. Teardown in the subscriber finalizer is the unsubscribe row.

Do not add a behavior that has no row. If the implementation needs a new variable, add it to the memory answer and add the rows that change it.

## Then turn the rows into tests

One row is one test. Arrange that memory, deliver that event, assert the next memory and the notifications, including silence. Use a `TestScheduler` when the event is a value, an error, a completion, or a tick. Assert unsubscribe, abort, or dispose directly when that is the action the row names.

## Worked example: latest-only gate

Policy: forward a source value only when a gate notifier has most recently emitted `true`. Values that arrive while the gate is closed are dropped, not stored. The gate starts closed. Gate errors and source errors fail the output. Source completion completes the output. Gate completion leaves the gate in its last position.

Memory: `closed`, `open`, and `finished`.

Memory at subscribe: `closed`. The gate notifier is subscribed as part of subscribe.

Events: source value, source error, source completion, gate value, gate error, gate completion, unsubscribe.

Notifications: source value, error, completion. Silence is allowed.

Rows that matter:

- A gate value `true` moves `closed` to `open` and sends nothing.
- A gate value `false` moves `open` to `closed` and sends nothing.
- A source value in `open` stays `open` and sends that value.
- A source value in `closed` stays `closed` and sends nothing.
- Source completion from either open or closed sends completion and finishes.
- Source error or gate error sends the error and finishes.
- Gate completion sends nothing and leaves the current memory.
- Unsubscribe finishes and unsubscribes both source and gate.
- A source value after `finished` sends nothing.

The sequence "source 1, gate true, source 2, gate false, source 3, source completion" sends `2` and then completion. The `1` and the `3` are the silence rows. An implementation that stored the latest closed value and flushed it on open would be a different machine; the checklist forbids that because no row says so.

## Contributor

SuperGrok (supergrok@x.ai) is the main contributor of this file.
