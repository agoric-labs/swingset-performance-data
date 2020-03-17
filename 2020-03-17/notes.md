Snapshot of performance benchmark data, 2020-03-17

Run with commit Agoric@9d8b545f6f8d0372bc83a33db7dc9d6ee87a8027

These data capture the state of affairs prior to beginning any of the work on
pruning resolved promises.

All of these were generated running the `swingset-runner` `demo/megaPong` test
using the command:

`time bin/runner --init ${db} --blockmode --logtimes --logmem ${gc} run demo/megaPong ${rounds}`

Guide to the data:

Stats File               |    `${db}` |     `${gc}` | `${rounds}` | `time` output
-------------------------|------------|-------------|-------------|--------------
`stats-filedb-100k-gc`   | `--filedb` | `--forcegc` |      100000 | `2298.487u 2201.175s 1:12:33.24 103.3%`
`stats-filedb-100k-nogc` | `--filedb` |             |      100000 | `1790.850u 2047.853s 1:03:17.56 101.0%`
`stats-filedb-10k-gc`    | `--filedb` | `--forcegc` |       10000 | `  43.387u   21.294s    0:59.38 108.9%`
`stats-filedb-10k-nogc`  | `--filedb` |             |       10000 | `  36.136u   21.860s    0:56.26 103.0%`
`stats-lmdb-100k-gc`     |   `--lmdb` | `--forcegc` |      100000 | ` 466.193u    5.894s    5:44.41 137.0%`
`stats-lmdb-100k-nogc`   |   `--lmdb` |             |      100000 | ` 188.540u    2.826s    3:10.06 100.6%`
`stats-lmdb-10k-gc`      |   `--lmdb` | `--forcegc` |       10000 | `  27.146u    0.456s    0:22.79 121.0%`
`stats-lmdb-10k-nogc`    |   `--lmdb` |             |       10000 | `  19.017u    0.423s    0:18.83 103.1%`
`stats-memdb-100k-gc`    |  `--memdb` | `--forcegc` |      100000 | ` 566.884u    6.904s    7:17.29 131.2%`
`stats-memdb-100k-nogc`  |  `--memdb` |             |      100000 | ` 187.825u    3.043s    3:09.02 100.9%`
`stats-memdb-10k-gc`     |  `--memdb` | `--forcegc` |       10000 | `  27.606u    0.413s    0:22.66 123.6%`
`stats-memdb-10k-nogc`   |  `--memdb` |             |       10000 | `  17.874u    0.348s    0:17.59 103.5%`
