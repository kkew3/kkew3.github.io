---
layout: post
title:  "使用 matplotlib 制作用于 macOS 的渐变色桌面"
date:   2024-02-01 11:22:51 +0800
tags:   os--macos misc dev--python
---

最近我喜欢上了纯色桌面，显得干净整洁。然而我发现一个问题，就是 Dock 在一些颜色下会变得不容易辨识。经过实验，Dock 在黑色下显得最清楚，但是使用纯黑色作桌面我感觉不是很美观。我希望有一个渐变色桌面，其中大部分是我想要的某种颜色，然后从偏底部至底部渐变为黑色，从而使 Dock 更清楚。然而 macOS 并没有提供这样的桌面。于是我决定使用 Python 的 [`matplotlib`](https://matplotlib.org/) 自己画一个这样的桌面。

思路是：

1. 使用 `system_profiler SPDisplaysDataType | grep Resolution` 获取屏幕的像素上的长宽；
2. 使用 `matplotlib.pyplot.cm.colors.LinearSegmentedColormap` 制作一个由我想要的颜色渐变为黑色的 colormap；
3. 构造一个以第 1 步为长宽、以第 2 步为 colormap 的矩阵，使其颜色满足上述渐变色要求；
4. 保存为图片。

主要问题出在第 4 步。我先去掉坐标轴，以为就没问题了，然而之后发现保存的图总是有一圈白色边框，怎么都去不掉（我尝试了[这个问题](https://stackoverflow.com/q/37809697/7881370)下的若干评论）。最终我采用了[这个回答](https://stackoverflow.com/a/37812313/7881370)的写法，虽然并不清楚原理 😅。总之问题就算解决了吧。

完整代码如下：

```python
#!/usr/bin/env python3
import argparse
from pathlib import Path
import subprocess
import re

import numpy as np
import matplotlib

matplotlib.use('Agg')
from matplotlib import pyplot as plt


def generate_wallpaper(
    name: str,
    primary_color_rgb,
    start_fade_position: float,
    force_save: bool,
):
    """
    Save faded color as wallpaper.

    :param name: the name to save
    :param primary_color_rgb: the RGB 3-tuple of uint8 value range
    :param start_fade_position: the position to start fading
    :param force_save: ``True`` to overwrite existing files
    """
    whs = []

    # 第 1 步，获取屏幕长宽
    proc = subprocess.run(['system_profiler', 'SPDisplaysDataType'],
                          text=True,
                          capture_output=True,
                          check=True)
    for line in re.findall(r'(.*)\n', proc.stdout):
        m = re.search(r'Resolution: (\d+) x (\d+)', line)
        if m:
            whs.append((int(m.group(1)), int(m.group(2))))

    # 第 2 步，构造渐变色 colormap
    colors = [np.asarray(primary_color_rgb) / 255, np.zeros(3)]
    cmap = plt.cm.colors.LinearSegmentedColormap.from_list(
        'colormap', colors, N=256)
    for j, (w, h) in enumerate(whs, 1):
        # 第 3 步，构造矩阵
        image = np.zeros((h, w))
        start = int(h * start_fade_position)
        steps = h - start
        # 使用 linspace 构造渐变色
        image[start:] = np.linspace(0, 1, steps)[:, np.newaxis]

        # 这里是不知道为什么能 work 的部分
        sizes = image.shape[::-1]
        fig = plt.figure()
        fig.set_size_inches(1. * sizes[0] / sizes[1], 1, forward = False)
        ax = plt.Axes(fig, [0., 0., 1., 1.])
        ax.set_axis_off()
        fig.add_axes(ax)
        ax.imshow(image, cmap=cmap, aspect='auto')

        # 保存为图片
        tofile = Path(f'{name}_{j}.jpg')
        if not force_save and tofile.exists():
            raise FileExistsError
        fig.savefig(tofile, dpi=sizes[0])
        plt.close(fig)


# 一些命令行参数
def make_parser():
    parser = argparse.ArgumentParser(
        description='Generate faded color wallpaper.')
    parser.add_argument(
        '-n', '--name', help='default to "wallpaper"', default='wallpaper')
    parser.add_argument(
        '-c',
        '--color',
        nargs=3,
        metavar=('R', 'G', 'B'),
        type=int,
        help='default to black',
        default=[0, 0, 0])
    parser.add_argument(
        '-p',
        '--fade-start-position',
        type=float,
        help='default to 0.0',
        default=0.0)
    parser.add_argument(
        '-f',
        '--force',
        action='store_true',
        help='force overwrite existing files')
    return parser


def main():
    args = make_parser().parse_args()
    generate_wallpaper(args.name, args.color, args.fade_start_position,
                       args.force)


if __name__ == '__main__':
    main()
```

来跑一个试试：

```bash
# 以上代码保存为 wallpaper_gen.py
python3 wallpaper_gen.py -c 0 54 9 -p 0.7
```

生成的图如下：

![darkgreen](/assets/posts_imgs/2024-02-01/darkgreen.jpg)
