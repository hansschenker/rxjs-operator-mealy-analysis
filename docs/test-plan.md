# Test plan method

SuperGrok is the main contributor of this project.

A 6-tuple analysis is already a test plan. Each operator file under `test-plans/` turns that analysis into cases. This note says how to read a case and how to turn it into an RxJS test.

## What one case is

A case fixes the memory, delivers one event, and checks two results:

- the memory afterwards
- the notifications sent downstream

Sending nothing is a result. Sending a value and then completion is one result, not two tests. Unsubscribe, abort, inner subscribe, and resource dispose are assertions when the analysis names them as actions.

The six analysis answers map onto the test as follows.

Memory under test is the state space. The arrange step builds one of those memories, not a vague "the operator is running."

Memory at subscribe is the initial state. The first case subscribes and asserts that memory before any source notification, unless the analysis says subscribe itself sends something.

Events are the input alphabet. A case delivers exactly one of them: a value, an error, a completion, an unsubscribe, a timer tick, or an inner notification.

Possible notifications are the output alphabet. The assert step expects a word over that alphabet, including the empty word.

The transition row is the memory assertion. The output row is the notification assertion. They are the same case, checked twice. Splitting them in the plan only keeps the two promises visible.

## Rules that every plan includes

Subscribe, then check the initial memory.

For every transition row, arrange the named memory, deliver the named event, and assert the resulting memory.

For every output row, arrange that memory, deliver that event, and assert the downstream word.

After the operator has errored, completed, or been unsubscribed, deliver one further event. Memory stays finished and nothing is sent.

Every source edge in the analysis is its own case. Those are the rows marble diagrams usually skip: flush on completion, no flush on error, no subscribe for `take` of zero, two subscriptions for `partition`.

## How to encode a case

Use a `TestScheduler` marble test when the event is a value, an error, a completion, or a time tick. The input marble is the event. The expected marble is the output word. A frame is the tick. Assert unsubscribe with the subscription marble when the analysis says the source is cancelled.

Assert a second subscription, an abort, or a disposed resource directly when the interesting result is not a notification. `partition` must show two source subscriptions unless the caller shares. `ajax` and `fromFetch` must abort on unsubscribe. `using` must dispose the resource.

One row is one test. A single test for the whole operator hides which promise failed.

## Worked reading

`pairwise`. After subscribe the memory is empty. The first value is stored and nothing is sent. The second value replaces the stored value and sends the pair. Completion sends completion and does not invent a pair. An error is forwarded. A value after completion is not sent.

`debounceTime`. A value stores the latest value, arms one task, and sends nothing. Another value before the tick replaces the stored value and does not replace the task. The tick sends the stored value if the due time has elapsed, otherwise it reschedules. Completion while holding sends the stored value and then completes. An error sends the error and does not flush. Unsubscribe clears the stored value so a late tick sends nothing.

`take`. Values before the count are sent and the count advances. The value that reaches the count is sent, then completion is sent, and the source is unsubscribed. `take` of zero completes and does not subscribe.

## Where the plans are

The index is [test-plans/README.md](../test-plans/README.md). Each file links back to its 6-tuple analysis.
