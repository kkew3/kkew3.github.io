---
layout: post
title:  "Stupidly Simple Way to Add Up Numbers in Shell"
date:   2025-10-21 10:38:49 +0800
tags:   dev--sh
---

Sometimes we need to add up numbers that come from stdin, one per line.
There are many ways to attack this problem.
To name a few:

- `awk '{sum += $_} END {print sum}'`;
- `perl -lne '$sum += $_; END {print $sum}'`;
- `python3 -c 'import sys; print(sum(map(int, sys.stdin)))'` (or run a float summation by changing `int` to `float`);
- `paste -sd+ - | bc`.

But today I [found](https://unix.stackexchange.com/a/249799) a stupidly simple and intuitive alternative: `jq -s add`, where `-s` treat the stdin as an array, and `add` add the elements in the array together.
Example:

```bash
shuf -e 5 7 | jq -s add
# output: 12
```

Note: we have used `shuf` (part of GNU coreutils) to get numbers printed to stdout, one per line, as the input.
Of course, we will need to have [`jq`](https://jqlang.org/) installed.

We may also extend the use case a bit to computing the mean.
For example:

```bash
shuf -e 5 7 | jq -s add/length
# output: 6
```
