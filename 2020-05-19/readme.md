Snapshot of performance benchmark data, 2020-05-19

Run with commit https://github.com/Agoric/agoric-sdk/commit/31c92bc45acf04089be32884bc62e6a3a8c20f28

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
`stats-lmdb-10k-gc`      | `--forcegc` |       10000 | ` 30.217u 0.670s 0:25.68 120.2%`
`stats-lmdb-10k-nogc`    |             |       10000 | ` 23.182u 0.681s 0:23.18 102.9%`
`stats-lmdb-100k-gc`     | `--forcegc` |      100000 | `291.187u 8.755s 4:09.26 120.3%`
`stats-lmdb-100k-nogc`   |             |      100000 | `209.806u 9.419s 3:40.23  99.5%`
