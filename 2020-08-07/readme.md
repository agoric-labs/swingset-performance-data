Snapshot of performance benchmark data, 2020-08-07

Run with commit https://github.com/Agoric/agoric-sdk/commit/184c9126d33a7f987d6d770df39416f0154e1045

These data capture measure the relative time costs of metering, now that
swingset-runner can run things in an environment that optionally supports
metering.

There are two stages of metering overhead:

- Instrumentation on the global objects -- this affects both the vats and the
kernel.  Note that the globals instrumentation has to be applied everwhere,
because of how JS works, but it is disabled while running in the kernel or in
unmetered vats.  However, code not subject to metering in this situation still
pays some overhead cost to have the globals instrumention present, because
the wrapping code still has to check whether it needs to meter or not.

- A metering transformation applied to dynamically loaded code -- this only
affects vats, and at that, only the vats for which it is indicated when those
vats were created.

Note that code subjected to the metering transform also requires instrumented
globals, so even though there are two different switches to turn on, there are
only three combinations: no metering, unmetered but with the globals metering
instrumentation enabled, and fully metered.  In production use we would only
expect the latter two states to actually appear, since we expect there always to
be some metered vats and this implies the enabling of the globals
instrumentation.  However, we want to _benchmark_ all three states in order to
find out how much the various layers of metering are costing us.

In addition, vats can be run from `swingset-runner` either directly or
indirectly.  In direct mode, `swingset-runner` itself instantiates a group of
statically defined vats and launches whichever one is designated the bootstrap
vat.  In indirect mode, `swingset-runner` instantiates and launches a launcher
vat, which is then passed a swingset config descriptor that _it_ uses to
dynamically instantiate the vats and launch their bootstrap vat.  This mechanism
exists because the metering transform can only be used on dynamic vats.  We
would like to compare metered and unmetered vat groups in conditions where the
only difference between them is the presence or absence of metering.  Thus we
end up with five cases total:

1. direct mode, no globals instrumentation (and consequently no metering either)
2. indirect mode, no globals instrumentation (and no metering)
3. direct mode, globals instrumentatation enabled (but no metering since it's not possible in this case)
4. indirect mode, globals instrumentatation enabled but no metering
5. indirect mode, fully metered

Note that as with metering, we don't expect some of these cases (notably 2 and
3) to ever really happen, but we include them in the benchmarks to measure the
differential effects of the various underlying options.

The timing numbers shown in the table below were generated using
`swingset-runner` running its `demo/exchangeBenchmark` benchmark.  This
benchmark has two vats, Alice and Bob, using the Zoe `simpleExchange` contract
to trade 1 simolean for 1 moola and then back again per benchmark round.

These tests were run using the command:

`time bin/runner --init --blockmode --blocksize 250 --logall --stats --benchmark ${rounds} ${flags} run demo/exchangeBenchmark`

The block size was upped to 250 cranks (from the default of 200) so that each
round would complete in a single block.  In this benchmark, rounds took 203-217
cranks, so capturing the whole round in one block makes the collected stats
records quite a bit cleaner but doesn't change any of the underlying analysis.

The benchmarks were run with 100 rounds (to get a sense of how
repetition affected things) and 1 round (to get a sense of the cost of
setup).

The choice of which of the five different possible metering options to run is
governed by the command line arguments denoted by `${flags}` above.  The
settings and their meanings are as follows:


Stats File                     | `${flags}`                   | Meaning
:------------------------------|:-----------------------------|:-------
`stats-no-metering-*`          |(empty)                       | direct mode, no metering
`stats-no-metering-indirect-*` |`--indirect`                  | indirect mode, no metering
`stats-globals-only-*`         |`--globalmetering`            | direct mode, globals instrumentation enabled
`stats-globals-indirect-*`     |`--globalmetering --indirect` | indirect mode, globals instrumentation enabled but no metering
`stats-full-metering-*`        |`--metering`                  | indirect mode with full metering

### Timings

Stats File                       | Rounds | `time` output
:--------------------------------|-------:|-------------:
`stats-no-metering-100`          |     100|`159.430u 4.248s 2:20.02 116.8%`
`stats-no-metering-1`            |       1|`  7.884u 0.466s 0:05.89 141.5%`
`stats-no-metering-indirect-100` |     100|`161.715u 4.403s 2:21.25 117.6%`
`stats-no-metering-indirect-1`   |       1|`  7.855u 0.454s 0:05.82 142.6%`
`stats-globals-only-100`         |     100|`159.466u 4.354s 2:20.17 116.8%`
`stats-globals-only-1`           |       1|`  8.103u 0.495s 0:06.00 143.1%`
`stats-globals-indirect-100`     |     100|`161.707u 4.270s 2:21.09 117.6%`
`stats-globals-indirect-1`       |       1|`  7.881u 0.498s 0:05.88 142.3%`
`stats-full-metering-100`        |     100|`162.538u 4.277s 2:22.90 116.7%`
`stats-full-metering-1`          |       1|`  9.215u 0.481s 0:06.86 141.2%`

### Results `swingset-runner` demos

Another set of timings ran each of the current set of
`swingset-runner` demo vat groups with the same set of conditions.

a) no meter, direct

b) no meter, indirect

c) globals instrumentation enabled, direct

d) globals instrumentation enabled, indirect

e) metered

- encouragementBot
   * a) ` 3.807u 0.313s 0:02.88 142.7%`
   * b) ` 4.004u 0.308s 0:03.01 142.8%`
   * c) ` 4.493u 0.304s 0:03.38 141.7%`
   * d) ` 4.686u 0.317s 0:03.53 141.3%`
   * e) ` 7.021u 0.339s 0:05.47 134.3%`
- exchangeBenchmark
   * a) ` 7.778u 0.486s 0:05.78 142.7%`
   * b) ` 8.994u 0.537s 0:06.72 141.6%`
   * c) `26.867u 0.557s 0:23.77 115.3%`
   * d) `26.061u 0.516s 0:22.95 115.7%`
   * e) `44.888u 0.840s 0:40.92 111.7%`
- justReply
   * a) ` 3.838u 0.312s 0:02.89 143.2%`
   * b) ` 4.444u 0.339s 0:03.30 144.5%`
   * c) ` 4.882u 0.324s 0:03.66 142.0%`
   * d) ` 4.606u 0.313s 0:03.46 141.9%`
   * e) ` 7.024u 0.344s 0:05.45 135.0%`
- megaPong
   * a) ` 4.078u 0.321s 0:03.01 145.8%`
   * b) ` 4.979u 0.394s 0:03.70 144.8%`
   * c) ` 5.155u 0.344s 0:03.87 141.8%`
   * d) ` 4.769u 0.320s 0:03.59 141.5%`
   * e) ` 8.239u 0.349s 0:06.57 130.4%`
- pingPongBenchmark
   * a) ` 3.934u 0.307s 0:02.93 144.3%`
   * b) ` 4.979u 0.389s 0:03.66 146.1%`
   * c) ` 5.006u 0.360s 0:03.79 141.4%`
   * d) ` 4.713u 0.309s 0:03.52 142.3%`
   * e) ` 8.032u 0.355s 0:06.37 131.5%`
- promiseChainBenchmark
   * a) ` 3.873u 0.315s 0:02.88 145.1%`
   * b) ` 4.865u 0.396s 0:03.71 141.5%`
   * c) ` 4.693u 0.336s 0:03.53 142.2%`
   * d) ` 4.491u 0.306s 0:03.38 141.7%`
   * e) ` 5.951u 0.324s 0:04.53 138.4%`
- resolveCase1
   * a) ` 3.755u 0.316s 0:02.84 142.9%`
   * b) ` 4.112u 0.322s 0:03.09 143.3%`
   * c) ` 4.537u 0.323s 0:03.44 140.9%`
   * d) ` 4.463u 0.310s 0:03.38 141.1%`
   * e) ` 5.922u 0.333s 0:04.50 138.8%`
- resolveCase2
   * a) ` 3.855u 0.314s 0:02.84 146.4%`
   * b) ` 4.187u 0.331s 0:03.12 144.5%`
   * c) ` 5.009u 0.346s 0:03.76 142.0%`
   * d) ` 4.474u 0.304s 0:03.36 141.9%`
   * e) ` 6.086u 0.333s 0:04.63 138.4%`
- resolveCase3
   * a) ` 3.851u 0.314s 0:02.88 144.4%`
   * b) ` 4.750u 0.398s 0:03.60 142.7%`
   * c) ` 4.760u 0.324s 0:03.56 142.6%`
   * d) ` 4.451u 0.301s 0:03.35 141.7%`
   * e) ` 5.950u 0.330s 0:04.51 139.2%`
- resolveChain
   * a) ` 3.879u 0.312s 0:02.90 144.1%`
   * b) ` 4.181u 0.328s 0:03.16 142.4%`
   * c) ` 4.676u 0.315s 0:03.53 141.0%`
   * d) ` 4.524u 0.306s 0:03.40 141.7%`
   * e) ` 5.961u 0.329s 0:04.54 138.3%`
- resolveCircular
   * a) ` 3.786u 0.313s 0:02.85 143.5%`
   * b) ` 3.973u 0.308s 0:02.94 145.2%`
   * c) ` 4.566u 0.321s 0:03.44 141.8%`
   * d) ` 4.483u 0.305s 0:03.37 141.8%`
   * e) ` 5.996u 0.328s 0:04.53 139.2%`
- resolveDelayedPipeline
   * a) ` 3.931u 0.304s 0:02.87 147.3%`
   * b) ` 4.773u 0.387s 0:03.61 142.6%`
   * c) ` 4.624u 0.327s 0:03.49 141.5%`
   * d) ` 4.513u 0.308s 0:03.40 141.4%`
   * e) ` 5.933u 0.329s 0:04.52 138.2%`
- resolveInArrayCase1
   * a) ` 3.833u 0.307s 0:02.84 145.4%`
   * b) ` 4.302u 0.348s 0:03.25 142.7%`
   * c) ` 4.352u 0.310s 0:03.29 141.6%`
   * d) ` 4.551u 0.306s 0:03.41 142.2%`
   * e) ` 5.894u 0.326s 0:04.48 138.6%`
- resolveIndirect
   * a) ` 3.782u 0.307s 0:02.85 143.1%`
   * b) ` 4.027u 0.330s 0:03.01 144.5%`
   * c) ` 4.416u 0.302s 0:03.33 141.4%`
   * d) ` 4.496u 0.305s 0:03.38 141.7%`
   * e) ` 5.930u 0.325s 0:04.51 138.5%`
- resolvePassResult
   * a) ` 4.064u 0.325s 0:02.99 146.4%`
   * b) ` 4.491u 0.335s 0:03.27 147.4%`
   * c) ` 4.384u 0.302s 0:03.29 142.2%`
   * d) ` 4.474u 0.308s 0:03.35 142.3%`
   * e) ` 5.928u 0.332s 0:04.53 137.9%`
- resolvePassResultPromise
   * a) ` 3.819u 0.307s 0:02.84 144.7%`
   * b) ` 4.276u 0.331s 0:03.14 146.4%`
   * c) ` 4.358u 0.309s 0:03.29 141.3%`
   * d) ` 4.507u 0.312s 0:03.42 140.6%`
   * e) ` 5.916u 0.326s 0:04.51 138.1%`
- resolvePipelined
   * a) ` 3.737u 0.313s 0:02.83 142.7%`
   * b) ` 4.209u 0.340s 0:03.17 143.2%`
   * c) ` 4.439u 0.306s 0:03.31 142.9%`
   * d) ` 4.478u 0.308s 0:03.39 140.7%`
   * e) ` 5.959u 0.326s 0:04.53 138.4%`
- resolvePromiseComplicated
   * a) ` 3.906u 0.306s 0:02.89 145.3%`
   * b) ` 4.044u 0.321s 0:03.08 141.5%`
   * c) ` 4.467u 0.303s 0:03.37 141.2%`
   * d) ` 4.768u 0.319s 0:03.57 142.0%`
   * e) ` 7.151u 0.342s 0:05.54 135.1%`
- resolveSimpleCicular
   * a) ` 3.818u 0.307s 0:02.83 145.2%`
   * b) ` 3.996u 0.309s 0:02.95 145.4%`
   * c) ` 4.399u 0.299s 0:03.30 141.8%`
   * d) ` 4.612u 0.313s 0:03.47 141.7%`
   * e) ` 5.875u 0.326s 0:04.48 138.1%`
- resolveWithEmbeddedPromise
   * a) ` 3.806u 0.312s 0:02.86 143.7%`
   * b) ` 3.916u 0.306s 0:02.93 143.6%`
   * c) ` 4.509u 0.303s 0:03.36 142.8%`
   * d) ` 4.520u 0.313s 0:03.40 142.0%`
   * e) ` 6.008u 0.324s 0:04.56 138.5%`
- simplePromisePass
   * a) ` 3.834u 0.318s 0:02.89 143.2%`
   * b) ` 4.000u 0.310s 0:02.96 145.6%`
   * c) ` 4.521u 0.317s 0:03.41 141.6%`
   * d) ` 4.623u 0.307s 0:03.46 142.1%`
   * e) ` 6.978u 0.338s 0:05.40 135.1%`
- vatResolveTest
   * a) ` 3.936u 0.311s 0:02.91 145.7%`
   * b) ` 3.966u 0.305s 0:02.96 143.9%`
   * c) ` 4.562u 0.315s 0:03.44 141.5%`
   * d) ` 4.822u 0.328s 0:03.63 141.5%`
   * e) ` 6.989u 0.340s 0:05.42 135.0%`
- vatResolveTestSimple
   * a) ` 3.928u 0.312s 0:02.91 145.3%`
   * b) ` 4.014u 0.308s 0:02.97 145.1%`
   * c) ` 4.529u 0.312s 0:03.40 142.0%`
   * d) ` 4.687u 0.324s 0:03.51 142.4%`
   * e) ` 7.019u 0.340s 0:05.43 135.3%`
- zoeTests simpleExchangeOk
   * a) ` 8.356u 0.528s 0:06.21 142.8%`
   * b) ` 8.453u 0.489s 0:06.31 141.5%`
   * c) `17.953u 0.514s 0:15.51 119.0%`
   * d) `18.152u 0.523s 0:15.61 119.6%`
   * e) `48.826u 1.081s 0:44.45 112.2%`
- zoeTests simpleExchangeNotifier
   * a) ` 8.296u 0.478s 0:06.25 140.1%`
   * b) ` 8.663u 0.533s 0:06.50 141.3%`
   * c) `17.851u 0.523s 0:15.33 119.8%`
   * d) `18.250u 0.526s 0:15.72 119.4%`
   * e) `49.637u 1.068s 0:45.11 112.3%`
- zoeTests autoswapOk
   * a) ` 8.371u 0.486s 0:06.29 140.6%`
   * b) ` 8.700u 0.465s 0:06.51 140.7%`
   * c) `18.953u 0.522s 0:16.41 118.6%`
   * d) `19.073u 0.528s 0:16.54 118.4%`
   * e) `51.357u 1.064s 0:47.03 111.4%`
- zoeTests automaticRefundOk
   * a) ` 8.113u 0.475s 0:06.08 141.1%`
   * b) ` 8.341u 0.518s 0:06.32 140.0%`
   * c) `14.319u 0.500s 0:11.94 124.0%`
   * d) `14.273u 0.504s 0:11.88 124.3%`
   * e) `44.938u 1.002s 0:40.72 112.7%`
- zoeTests coveredCallOk
   * a) ` 8.410u 0.514s 0:06.35 140.4%`
   * b) ` 8.702u 0.484s 0:06.56 139.9%`
   * c) `17.458u 0.494s 0:15.03 119.3%`
   * d) `17.586u 0.516s 0:15.08 119.9%`
   * e) `49.674u 1.062s 0:45.19 112.2%`
- zoeTests swapForOptionOk
   * a) ` 9.512u 0.527s 0:07.13 140.6%`
   * b) ` 9.612u 0.552s 0:07.24 140.3%`
   * c) `26.476u 0.566s 0:23.47 115.1%`
   * d) `27.139u 0.594s 0:24.00 115.5%`
   * e) `59.877u 1.227s 0:54.93 111.2%`
- zoeTests publicAuctionOk
   * a) ` 8.594u 0.482s 0:06.48 139.9%`
   * b) ` 8.604u 0.483s 0:06.49 139.9%`
   * c) `17.588u 0.503s 0:15.13 119.4%`
   * d) `17.595u 0.518s 0:15.06 120.1%`
   * e) `51.365u 1.098s 0:46.81 112.0%`
- zoeTests atomicSwapOk
   * a) ` 8.514u 0.488s 0:06.43 139.8%`
   * b) ` 8.349u 0.470s 0:06.29 140.0%`
   * c) `17.293u 0.515s 0:14.80 120.2%`
   * d) `17.482u 0.517s 0:14.97 120.1%`
   * e) `50.030u 1.066s 0:45.59 112.0%`
- zoeTests simpleExchangeOk
   * a) ` 8.782u 0.519s 0:06.61 140.5%`
   * b) ` 9.036u 0.540s 0:06.77 141.3%`
   * c) `17.792u 0.522s 0:15.26 119.9%`
   * d) `17.994u 0.524s 0:15.53 119.1%`
   * e) `49.160u 1.082s 0:44.88 111.9%`
- zoeTests simpleExchangeNotifier
   * a) ` 8.724u 0.535s 0:06.56 141.0%`
   * b) ` 8.586u 0.469s 0:06.47 139.7%`
   * c) `17.949u 0.525s 0:15.44 119.5%`
   * d) `18.177u 0.509s 0:15.72 118.7%`
   * e) `49.408u 1.068s 0:45.00 112.1%`
- zoeTests autoswapOk
   * a) ` 9.009u 0.548s 0:06.71 142.1%`
   * b) ` 8.657u 0.530s 0:06.47 141.8%`
   * c) `19.499u 0.528s 0:16.97 117.9%`
   * d) `19.249u 0.528s 0:16.71 118.2%`
   * e) `53.096u 1.092s 0:48.53 111.6%`
- zoeTests sellTicketsOk
   * a) ` 8.916u 0.501s 0:06.70 140.4%`
   * b) ` 9.388u 0.530s 0:07.03 140.9%`
   * c) `26.915u 0.565s 0:23.95 114.6%`
   * d) `27.309u 0.582s 0:24.24 115.0%`
   * e) `59.621u 1.141s 0:54.81 110.8%`

