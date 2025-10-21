---
layout: post
title:  "Filter Text File In-place with Vim Ex Mode"
date:   2025-10-21 11:07:26 +0800
tags:   editor--vim
---

We sometimes need to filter a text file (denoted as `file.txt` in this post) using some external program in-place, e.g. pretty-formatting a compact json file using [`jq`](https://jqlang.org/).
By "filter in-place", I mean to pass the text file to the external program as stdin, and overwrite the original file with the program's stdout.
A direct solution is:

```bash
tmp=$(mktemp tmp.XXXXXX) && jq < file.txt > $tmp && mv $tmp file.txt
```

Inspired by [this](https://vi.stackexchange.com/a/2692/48634) post, I found a more concise way to implement the function:

```bash
vim -es -c '%!jq' -cx file.txt
```

Explanation:

- `vim -es` starts `vim` in Ex mode (`-e`) silently (`-s`).
- `-c '%!jq'` filters (`!`) the entire (`%`) buffer using `jq`.
- `-cx` saves the buffer and exits (see [vim-help](https://vimhelp.org/editing.txt.html#%3Ax) for more about `:x` command).
