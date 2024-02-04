---
layout: post
title:  "Host Python packages with Jekyll on GitHub Pages"
date:   2024-02-04 17:31:26 +0800
tags:   dev--python
---

I have a collection of Python packages, either hosted locally or on GitHub, dedicated to my own use, and I don't feel like uploading them to [PyPI](https://pypi.org/).
However, it soon becomes a pain when I need to install one of them to my virtual environment, since they all scatter about my disk, and I have to `cd` to the corresponding project directory and do `pip install .`.
It would be preferable to stay in my current repository, and do `pip install ...`.
If the package is already hosted on GitHub, like [alfred_fzf_helper](https://github.com/kkew3/alfred_fzf_helper), I may do `pip install git+https://github.com/kkew3/alfred_fzf_helper.git` directly.
This is not good enough, since I still need to memorize the URL, and it's not convenient, if not impossible, to specify the version requirements.

Luckily, hosting a private Python package repository is possible, and freely available with Jekyll and GitHub Pages.
Following [this guide](https://packaging.python.org/en/latest/guides/hosting-your-own-index/), after making a directory `pip` under the root of my site, I put my Python source distribution tarballs into it.
After some googling, I find that Jekyll does not support autoindexing out-of-the-box.
If I push the tarballs onto GitHub, `pip` won't be able to find the source distributions.

I will exploit the `--find-links` option of `pip install` instead.
What we need, then, is simply an HTML page that lists all the URLs to the tarballs hosted.
With simple Liquid, I loop over all static files under `pip` directory and list them in an unordered list:

{% raw %}
```liquid
---
layout: default
---

<h1>Index of {{ page.path }}</h1>
<ul>
  {% assign pip_packages = site.static_files | where: "pip_package", true %}
  {% for item in pip_packages %}
    <li><a href="{{ site.baseurl }}{{ item.path }}">{{ item.path }}</a></li>
  {% endfor %}
</ul>
```
{% endraw %}

where `pip_package` is defined in `_config.yml` like this (see [here](https://jekyllrb.com/docs/static-files/#add-front-matter-to-static-files) for more details):

```yaml
defaults:
  - scope:
      path: "pip"
    values:
      pip_package: true
```

Finally, I insert the following lines to `~/.config/pip/pip.conf`:

```ini
[install]
find-links = https://kkew3.github.io/pip
```

To check if it works, create a virtual environment (omitted below) and install one of the hosted package:

```bash
pip install "alfred-fzf-helper>=0.2"
```

It works!
