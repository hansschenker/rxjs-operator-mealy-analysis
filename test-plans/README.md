# Test plans

SuperGrok is the main contributor of this project.

Each plan is derived from the matching 6-tuple analysis. One row of memory plus event is one test. Silence is an expected result.

## conditional

- [defaultIfEmpty](test-plans/conditional/defaultIfEmpty.md)
- [every](test-plans/conditional/every.md)
- [find](test-plans/conditional/find.md)
- [findIndex](test-plans/conditional/findIndex.md)
- [isEmpty](test-plans/conditional/isEmpty.md)
- [sequenceEqual](test-plans/conditional/sequenceEqual.md)

## creation

- [ajax](test-plans/creation/ajax.md)
- [animationFrames](test-plans/creation/animationFrames.md)
- [bindCallback](test-plans/creation/bindCallback.md)
- [bindNodeCallback](test-plans/creation/bindNodeCallback.md)
- [defer](test-plans/creation/defer.md)
- [empty](test-plans/creation/empty.md)
- [from](test-plans/creation/from.md)
- [fromEvent](test-plans/creation/fromEvent.md)
- [fromEventPattern](test-plans/creation/fromEventPattern.md)
- [fromFetch](test-plans/creation/fromFetch.md)
- [generate](test-plans/creation/generate.md)
- [iif](test-plans/creation/iif.md)
- [interval](test-plans/creation/interval.md)
- [never](test-plans/creation/never.md)
- [of](test-plans/creation/of.md)
- [pairs](test-plans/creation/pairs.md)
- [range](test-plans/creation/range.md)
- [throwError](test-plans/creation/throwError.md)
- [timer](test-plans/creation/timer.md)
- [using](test-plans/creation/using.md)
- [webSocket](test-plans/creation/webSocket.md)

## error-handling

- [catchError](test-plans/error-handling/catchError.md)
- [onErrorResumeNext](test-plans/error-handling/onErrorResumeNext.md)
- [onErrorResumeNextWith](test-plans/error-handling/onErrorResumeNextWith.md)
- [retry](test-plans/error-handling/retry.md)
- [retryWhen](test-plans/error-handling/retryWhen.md)
- [throwIfEmpty](test-plans/error-handling/throwIfEmpty.md)

## filtering

- [audit](test-plans/filtering/audit.md)
- [auditTime](test-plans/filtering/auditTime.md)
- [debounce](test-plans/filtering/debounce.md)
- [debounceTime](test-plans/filtering/debounceTime.md)
- [distinct](test-plans/filtering/distinct.md)
- [distinctUntilChanged](test-plans/filtering/distinctUntilChanged.md)
- [distinctUntilKeyChanged](test-plans/filtering/distinctUntilKeyChanged.md)
- [elementAt](test-plans/filtering/elementAt.md)
- [filter](test-plans/filtering/filter.md)
- [first](test-plans/filtering/first.md)
- [ignoreElements](test-plans/filtering/ignoreElements.md)
- [last](test-plans/filtering/last.md)
- [sample](test-plans/filtering/sample.md)
- [sampleTime](test-plans/filtering/sampleTime.md)
- [single](test-plans/filtering/single.md)
- [skip](test-plans/filtering/skip.md)
- [skipLast](test-plans/filtering/skipLast.md)
- [skipUntil](test-plans/filtering/skipUntil.md)
- [skipWhile](test-plans/filtering/skipWhile.md)
- [take](test-plans/filtering/take.md)
- [takeLast](test-plans/filtering/takeLast.md)
- [takeUntil](test-plans/filtering/takeUntil.md)
- [takeWhile](test-plans/filtering/takeWhile.md)
- [throttle](test-plans/filtering/throttle.md)
- [throttleTime](test-plans/filtering/throttleTime.md)

## join

- [combineAll](test-plans/join/combineAll.md)
- [combineLatestAll](test-plans/join/combineLatestAll.md)
- [combineLatestWith](test-plans/join/combineLatestWith.md)
- [concatAll](test-plans/join/concatAll.md)
- [concatWith](test-plans/join/concatWith.md)
- [endWith](test-plans/join/endWith.md)
- [exhaustAll](test-plans/join/exhaustAll.md)
- [mergeAll](test-plans/join/mergeAll.md)
- [mergeWith](test-plans/join/mergeWith.md)
- [raceWith](test-plans/join/raceWith.md)
- [startWith](test-plans/join/startWith.md)
- [switchAll](test-plans/join/switchAll.md)
- [withLatestFrom](test-plans/join/withLatestFrom.md)
- [zipAll](test-plans/join/zipAll.md)
- [zipWith](test-plans/join/zipWith.md)

## join-creation

- [combineLatest](test-plans/join-creation/combineLatest.md)
- [concat](test-plans/join-creation/concat.md)
- [forkJoin](test-plans/join-creation/forkJoin.md)
- [merge](test-plans/join-creation/merge.md)
- [partition](test-plans/join-creation/partition.md)
- [race](test-plans/join-creation/race.md)
- [zip](test-plans/join-creation/zip.md)

## mathematical

- [count](test-plans/mathematical/count.md)
- [max](test-plans/mathematical/max.md)
- [min](test-plans/mathematical/min.md)
- [reduce](test-plans/mathematical/reduce.md)

## multicasting

- [connect](test-plans/multicasting/connect.md)
- [multicast](test-plans/multicasting/multicast.md)
- [publish](test-plans/multicasting/publish.md)
- [publishBehavior](test-plans/multicasting/publishBehavior.md)
- [publishLast](test-plans/multicasting/publishLast.md)
- [publishReplay](test-plans/multicasting/publishReplay.md)
- [refCount](test-plans/multicasting/refCount.md)
- [share](test-plans/multicasting/share.md)
- [shareReplay](test-plans/multicasting/shareReplay.md)

## transformation

- [buffer](test-plans/transformation/buffer.md)
- [bufferCount](test-plans/transformation/bufferCount.md)
- [bufferTime](test-plans/transformation/bufferTime.md)
- [bufferToggle](test-plans/transformation/bufferToggle.md)
- [bufferWhen](test-plans/transformation/bufferWhen.md)
- [concatMap](test-plans/transformation/concatMap.md)
- [concatMapTo](test-plans/transformation/concatMapTo.md)
- [exhaust](test-plans/transformation/exhaust.md)
- [exhaustMap](test-plans/transformation/exhaustMap.md)
- [expand](test-plans/transformation/expand.md)
- [flatMap](test-plans/transformation/flatMap.md)
- [groupBy](test-plans/transformation/groupBy.md)
- [map](test-plans/transformation/map.md)
- [mapTo](test-plans/transformation/mapTo.md)
- [mergeMap](test-plans/transformation/mergeMap.md)
- [mergeMapTo](test-plans/transformation/mergeMapTo.md)
- [mergeScan](test-plans/transformation/mergeScan.md)
- [pairwise](test-plans/transformation/pairwise.md)
- [pluck](test-plans/transformation/pluck.md)
- [scan](test-plans/transformation/scan.md)
- [switchMap](test-plans/transformation/switchMap.md)
- [switchMapTo](test-plans/transformation/switchMapTo.md)
- [switchScan](test-plans/transformation/switchScan.md)
- [window](test-plans/transformation/window.md)
- [windowCount](test-plans/transformation/windowCount.md)
- [windowTime](test-plans/transformation/windowTime.md)
- [windowToggle](test-plans/transformation/windowToggle.md)
- [windowWhen](test-plans/transformation/windowWhen.md)

## utility

- [delay](test-plans/utility/delay.md)
- [delayWhen](test-plans/utility/delayWhen.md)
- [dematerialize](test-plans/utility/dematerialize.md)
- [finalize](test-plans/utility/finalize.md)
- [materialize](test-plans/utility/materialize.md)
- [observeOn](test-plans/utility/observeOn.md)
- [repeat](test-plans/utility/repeat.md)
- [repeatWhen](test-plans/utility/repeatWhen.md)
- [subscribeOn](test-plans/utility/subscribeOn.md)
- [tap](test-plans/utility/tap.md)
- [timeInterval](test-plans/utility/timeInterval.md)
- [timeout](test-plans/utility/timeout.md)
- [timeoutWith](test-plans/utility/timeoutWith.md)
- [timestamp](test-plans/utility/timestamp.md)
- [toArray](test-plans/utility/toArray.md)

