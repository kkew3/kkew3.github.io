---
layout: post
title:  "Poor man's parallel gzip"
date:   2025-07-15 17:52:27 +0800
tags:   dev--sh
---

There have been several tools aiming to incorporate multiple CPU cores to accelerate large file compression.
To name a few: [`pigz`](https://zlib.net/pigz/), [`zstd`](https://facebook.github.io/zstd/), [`xz`](https://tukaani.org/xz/).
Here I suggest a minimalist approach to parallel compression.
The core idea:

1. Split a file into chunks.
2. Compress the chunks in parallel.
3. Bundle the results in a tarball.

This approach is remarkably easy to implement and surprisingly effective for many use-cases—especially when you just want "good enough" speed without extra software.

Here're the hackable shell functions that implement compression and decompression:

```bash
# Write the tarball containing gzipped chunks to stdout.
# Usage: cat FILE | pgzip > OUTPUT.tar
pgzip() {
    local workdir="$(mktemp -d "${TMPDIR:-/tmp}/tmpXXXXXX")"
    if [ $? -ne 0 ]; then
        echo "Failed to mktemp -d" >&2
        return 1
    fi

    # Some fancy strategies to decide the number of chunks to split.
    local ncpu="$(nproc)"  # Get the number of processors.
    ncpu=$(( $ncpu * 8 / 10 ))  # Use 80% of all processors.
    ncpu=$(( $ncpu < 10 ? $ncpu : 10 ))  # Use at most 10 processors.

    # Step 1: split stdin into chunks.
    split -d -a 1 -n $ncpu - "$workdir/chunk_"
    # Step 2: parallel gzip.
    find "$workdir" -name 'chunk_*' | xargs -n1 -P $ncpu gzip
    # Step 3: bundle into tarball, which goes to stdout.
    { echo "-C $workdir"; find "$workdir" -name 'chunk_*' | xargs basename | sort; } | tar cf - -T -
    rm -rf "$workdir"
}

# Decompress the output tarball to stdout.
# Usage: pgunzip OUTPUT.tar > FILE
pgunzip() {
    local INPUT="$1"
    if [ -z "$INPUT" ]; then
        echo "Missing input file" >&2
        return 1
    fi
    tar xOf "$INPUT" | gzip -cd
}
```

That’s it. No compilation, no installation, just shell and standard tools.

What's more, the result tarball can be easily handled by python, with builtin modules only (i.e. `tarfile` and `gzip`).

The limitation is obvious:

- Not robust against failures mid-process.
