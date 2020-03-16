# swingset-performance-data

This repository is a place to accumulate data captured while working
on SwingSet kernel performance.  As we measure speed and memory usage,
we generate data files that can be graphed, studied, etc., but in
order to tell if we've actually succeeded in improving things we need
to compare before and after.  So we need some place to put the data
while the code itself is in flux.  This is that place.

Currently, each directory in this repo is a snapshot of a set of
benchmarks collected under some circumstances of interest (typically
the data associated with some version of the code) along with
annotation files describing what's there, why it's there, what commit
hash of the code the numbers are pertinent to, etc.  Right now it's a
pretty loose structure, though we expect it's likely to evolve and
become more codified with time.  But it's better than storing these
files in some working directory on somebody's laptop, which is where
they'd end up otherwise.
