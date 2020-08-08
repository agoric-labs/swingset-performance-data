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

1 direct mode, no globals instrumentation (and consequently no metering either)
2 indirect mode, no globals instrumentation (and no metering)
3 direct mode, globals instrumentatation enabled (but no metering since it's not possible in this case)
4 indirect mode, globals instrumentatation enabled but no metering
5 indirect mode, fully metered

Note that as with metering, we don't expect some of these cases (notably 2 and
3) to ever really happen, but we include them in the benchmarks to measure the
differential effects of the various underlying options.

The timing numbers shown in the table below were generated using
`swingset-runner` running its `demo/exchangeBenchmark` benchmark.  This
benchmark has two vats, Alice and Bob, using the Zoe `simpleExchange` contract
to trade 1 simolean for 1 moola and then back again per benchmark round.

These tests were run using the command:

`time bin/runner --init --blockmode --blocksize 250 --logall --stats --benchmark 100 ${flags} run demo/exchangeBenchmark`

The block size was upped to 250 cranks (from the default of 200) so that each
round would complete in a single block.  In this benchmark, rounds took 203-217
cranks, so capturing the whole round in one block makes the collected stats
records quite a bit cleaner but doesn't change any of the underlying analysis.

The choice of which of the five different possible metering options to run is
governed by the command line arguments denoted by `${flags}` above.  The
settings and their meanings are as follows:


Stats File                   | `${flags}`                   | Meaning
:----------------------------|:-----------------------------|:-------
`stats-no-metering`          |(empty)                       | direct mode, no metering
`stats-no-metering-indirect` |`--indirect`                  | indirect mode, no metering
`stats-globals-only`         |`--globalmetering`            | direct mode, globals instrumentation enabled
`stats-globals-indirect`     |`--globalmetering --indirect` | indirect mode, globals instrumentation enabled but no metering
`stats-full-metering`        |`--metering`                  | indirect mode with full metering

### Timings

Stats File                   | `time` output
:----------------------------|:-------------
`stats-no-metering`          |`159.430u 4.248s 2:20.02 116.8%`
`stats-no-metering-indirect` |`161.715u 4.403s 2:21.25 117.6%`
`stats-globals-only`         |`159.466u 4.354s 2:20.17 116.8%`
`stats-globals-indirect`     |`161.707u 4.270s 2:21.09 117.6%`
`stats-full-metering`        |`162.538u 4.277s 2:22.90 116.7%`
