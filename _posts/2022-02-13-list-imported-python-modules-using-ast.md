---
layout: post
title:  "使用抽象语法树ast统计哪些Python包与模块被导入了"
date:   2022-02-13 22:25:18 +0800
tags:   dev--python
---

长话短说，我的[Gist](https://gist.github.com/kkew3/3bd1e0255af63e3ac0801c7bbc89e7bf)。

给定一个没有`requirements.txt`的Python项目，如果想知道需要安装哪些包才能满足这个项目的依赖需求，一个容易想到的方法就是对每一个`.py`文件，用模式匹配（如正则表达式）找`import xxx`，并记录`xxx`为需要的包。然而`import`语句有很多形式，如：`import xxx`、`import xxx as aaa`、`import xxx as aaa, yyy as bbb`、`from xxx.yyy import fff as ccc`、`from .zzz import ggg`。因此，更好的方法是利用抽象语法树`ast`模块来找出所有`import`语句。

Python的`import`语句对应`ast`的两种节点：`ast.Import`和`ast.ImportFrom`。要从[`ast.Import`](https://docs.python.org/3/library/ast.html#ast.Import)获取导入包的列表，可用：

```python
[a.name for a in node.names]  # 其中node是ast.Import类型的
```

要从[`ast.ImportFrom`](https://docs.python.org/3/library/ast.html#ast.ImportFrom)获取导入的包，可用：

```python
node.module  # 其中node是ast.ImportFrom类型的
```

值得注意的是如果当前`import`语句是`from . import xxx`，`node.module`将会是`None`，此时`node.level > 0`，意味着相对导入。因此，要想获得所有导入的包（除了相对导入外，因为相对导入的包绝不会是需要安装的依赖），可以这样：

```python
import ast
# 假设source包含待解析源码
root = ast.parse(source)
result = []
for node in ast.walk(root):
    if isinstance(node, ast.Import):
        for a in node.names:
            result.append(a.name.split('.', maxsplit=1)[0])
    elif isinstance(node, ast.ImportFrom):
        if node.level == 0:
            result.append(node.module.split('.', maxsplit=1)[0])
```

然而绝对导入的包也有可能是工作目录中已存在的模块或包啊，此时我们就可以根据导入路径判断它是不是指工作目录下的包：

```python
def exists_local(path, rootpkg):
    filepath = os.path.join(rootpkg, path.replace('.', os.path.sep))
    # see if path is a local package
    if os.path.isdir(filepath) and os.path.isfile(
            os.path.join(filepath, '__init__.py')):
        return True
    # see if path is a local module
    if os.path.isfile(filepath + '.py'):
        return True

    return False
```

其中`path`是导入路径，`rootpkg`是根包所在目录（定义见[这里](https://docs.python.org/3.7/distutils/setupscript.html#listing-whole-packages)）。

把这个核心功能稍作包装，便可写出下面的完整可执行代码：

```python
from __future__ import print_function

import argparse
import os
import ast
import sys
import pkgutil
import itertools
import logging
import json


def make_parser():
    parser = argparse.ArgumentParser(
        description=('List all root imports. The *root* import of '
                     '`import pkg1.mod1` is "pkg1".'))
    parse_opts = parser.add_mutually_exclusive_group()
    parse_opts.add_argument(
        '-g',
        '--greedy',
        action='store_true',
        help=('find also import statements within try block, '
              'if block, while block, function definition, '
              'etc.'))
    parse_opts.add_argument(
        '-a',
        '--all',
        action='store_true',
        help=('first list all minimal-required root '
              'imports (without `-g\'), then list '
              'additionally-required root imports (with '
              '`-g\'), and explain the two lists'))
    parser.add_argument(
        '-i',
        '--include-installed',
        action='store_true',
        help='include installed/built-in modules/packages')
    parser.add_argument(
        '-T',
        '--files-from',
        metavar='LIST_FILE',
        help=('if specified, the files to process '
              'will be read one per line from '
              'LIST_FILE; if specified as `-\', '
              'stdin will be expected to contain '
              'the files to process. Note that '
              'SOURCE_FILEs, if exist, take '
              'precedence (see below)'))
    parser.add_argument(
        '--ipynb',
        action='store_true',
        help=('if specified, the files ending with '
              '".ipynb" in either SOURCE_FILEs or '
              'LIST_FILE will be parsed as ipython '
              'notebook files rather than Python '
              'files'))
    parser.add_argument(
        'rootpkg',
        metavar='ROOTPKG_DIR',
        type=dir_type,
        help=
        ('the directory of the root package. See '
         'https://docs.python.org/3.7/distutils/setupscript.html#listing-whole-packages '
         'about *root package*. Local packages/modules will be '
         'excluded from the results. For example, if '
         'there are "mod1.py" and "mod2.py", and in '
         '"mod2.py" there is `import mod1`, then "mod1" '
         'won\'t be listed in the result.'))
    parser.add_argument(
        'filenames',
        metavar='SOURCE_FILE',
        nargs='*',
        help=('if specified one or more files, '
              'only these SOURCE_FILEs will get '
              'processed regardless of `-T\' '
              'option; if no SOURCE_FILE is '
              'specified, `-T\', if exists, is '
              'processed. In both cases, the '
              'final results will be joined'))
    return parser


def dir_type(string):
    if not os.path.isdir(string):
        raise argparse.ArgumentTypeError('must be a directory')
    return string


# Reference: https://stackoverflow.com/a/9049549/7881370
def yield_imports(root, greedy):
    """
    Yield all absolute imports.
    """
    traverse = ast.walk if greedy else ast.iter_child_nodes
    for node in traverse(root):
        if isinstance(node, ast.Import):
            for a in node.names:
                yield a.name
        elif isinstance(node, ast.ImportFrom):
            # if node.level > 0, the import is relative
            if node.level == 0:
                yield node.module


def exists_local(path, rootpkg):
    """
    Returns ``True`` if the absolute import ``path`` refers to a package or
    a module residing under the working directory, else ``False``.
    """
    filepath = os.path.join(rootpkg, path.replace('.', os.path.sep))
    # see if path is a local package
    if os.path.isdir(filepath) and os.path.isfile(
            os.path.join(filepath, '__init__.py')):
        return True
    # see if path is a local module
    if os.path.isfile(filepath + '.py'):
        return True

    return False


def filter_local(imports_iterable, rootpkg):
    """
    Remove modules and packages in the working directory, and yield root
    imports.
    """
    for path in imports_iterable:
        if not exists_local(path, rootpkg):
            yield path.split('.', 1)[0]


def filter_installed(imports_iterable):
    """
    Remove modules and packages already installed, which include built-in
    modules and packages and those already installed (e.g. via ``pip``).
    """
    installed = set(
        itertools.chain(sys.builtin_module_names,
                        (x[1] for x in pkgutil.iter_modules())))
    for name in imports_iterable:
        if name not in installed:
            yield name


def collect_sources(filenames, files_from):
    if filenames:
        for filename in filenames:
            yield filename
    elif files_from == '-':
        try:
            for line in sys.stdin:
                yield line.rstrip('\n')
        except KeyboardInterrupt:
            pass
    elif files_from:
        try:
            with open(files_from) as infile:
                for line in infile:
                    yield line.rstrip('\n')
        except OSError:
            logging.exception('failed to read from "{}"'.format(files_from))


def parse_python(filename):
    with open(filename) as infile:
        root = ast.parse(infile.read(), filename)
    return root


def parse_ipynb(filename):
    source = []
    with open(filename) as infile:
        obj = json.load(infile)
    for c in obj['cells']:
        if c['cell_type'] == 'code':
            source.extend(map(str.rstrip, c['source']))
    source = (l for l in source if not l.lstrip().startswith('%'))
    source = '\n'.join(source)
    root = ast.parse(source, filename)
    return root


def produce_results(filenames, files_from, greedy, rootpkg, include_installed,
                    ipynb):
    all_imports = []
    for filename in collect_sources(filenames, files_from):
        parse_source = (parse_ipynb if ipynb and filename.endswith('.ipynb')
                        else parse_python)
        try:
            root = parse_source(filename)
        except OSError:
            logging.exception('skipped')
        except SyntaxError:
            logging.exception('failed to parse "{}"; skipped'.format(filename))
        else:
            all_imports.append(yield_imports(root, greedy))
    all_imports = itertools.chain.from_iterable(all_imports)
    all_imports = filter_local(all_imports, rootpkg)
    if not include_installed:
        all_imports = filter_installed(all_imports)
    all_imports = set(all_imports)
    return all_imports


def main():
    logging.basicConfig(format='%(levelname)s: %(message)s')
    args = make_parser().parse_args()

    if not args.all:
        all_imports = produce_results(args.filenames, args.files_from,
                                      args.greedy, args.rootpkg,
                                      args.include_installed, args.ipynb)
        if all_imports:
            print('\n'.join(sorted(all_imports)))
    else:
        min_imports = produce_results(args.filenames, args.files_from,
                                      False, args.rootpkg,
                                      args.include_installed, args.ipynb)
        max_imports = produce_results(args.filenames, args.files_from,
                                      True, args.rootpkg,
                                      args.include_installed, args.ipynb)
        extra_imports = max_imports - min_imports
        printed_min_imports = False
        if min_imports:
            print('# minimal imports:')
            print('\n'.join(sorted(min_imports)))
            printed_min_imports = True
        if extra_imports:
            # pretty formatting purpose
            if printed_min_imports:
                print()
            print('# additional possible imports:')
            print('\n'.join(sorted(extra_imports)))

    logging.shutdown()


if __name__ == '__main__':
    main()
```

需要注意的是，程序的输出并不一定是PyPI上包的名字（例如，`import bs4`然而`pip install beautifulsoup4`）。

---

类似项目：[pipreqs](https://github.com/bndr/pipreqs)。核心代码是几乎一样的，但包装得不同。
