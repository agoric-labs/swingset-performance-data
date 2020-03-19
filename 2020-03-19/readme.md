Snapshot of performance benchmark data, 2020-03-19

Run with commit https://github.com/Agoric/agoric-sdk/commit/03f106e6ee88855adccac4b3df8a06852cc03b16

These data capture the state of affairs after installing the code to prune
resolved promises in vats but prior to any of the corresponding changes in the
kernel itself (tracking via issue [#675](https://github.com/Agoric/agoric-sdk/issues/675))

All of these were generated running the `swingset-runner` `demo/megaPong` test
using the command:

`time bin/runner --init --blockmode --logall ${gc} run demo/megaPong ${rounds}`

Now that `--lmdb` is the chosen database (for the time being), we're no longer
bothering with capturing stats for `--filedb` or `--memdb`.  They have been
retained as developmental tools but are no longer relevant to the performance
domain.

Guide to the data:

Stats File               |     `${gc}` | `${rounds}` | `time` output
:------------------------|:------------|------------:|:-------------
`stats-lmdb-10k-gc`      | `--forcegc` |       10000 | ` 24.306u 0.437s 0:21.11 117.1%`
`stats-lmdb-10k-nogc`    |             |       10000 | ` 19.058u 0.492s 0:18.96 103.0%`
`stats-lmdb-100k-gc`     | `--forcegc` |      100000 | `224.492u 3.348s 3:16.06 116.2%`
`stats-lmdb-100k-nogc`   |             |      100000 | `172.203u 3.471s 2:56.05  99.7%`
