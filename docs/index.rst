欢迎查阅 Click 中文文档
==================================

Click 是一个利用很少的代码以可组合的方式创造优雅命令行工具接口的 Python 库。 它是高度可配置的，但却有合理默认值的“命令行接口创建工具”。

它致力于将创建命令行工具的过程变的快速而有趣，免除你因无法实现一个 CLI API 的挫败感。

Click 的三个特性:

-   任意嵌套命令
-   自动生成帮助页面
-   支持在运行时延迟加载子命令

那么它到底什么样呢?  下面有一个简单的 Click 项目例子:

.. click:example::

    import click

    @click.command()
    @click.option('--count', default=1, help='Number of greetings.')
    @click.option('--name', prompt='Your name',
                  help='The person to greet.')
    def hello(count, name):
        """Simple program that greets NAME for a total of COUNT times."""
        for x in range(count):
            click.echo('Hello %s!' % name)

    if __name__ == '__main__':
        hello()

当它运行的时候是这样的:

.. click:run::

    invoke(hello, ['--count=3'], prog_name='python hello.py', input='John\n')

它会自动生成美观的格式化帮助页面：

.. click:run::

    invoke(hello, ['--help'], prog_name='python hello.py')

你可以通过 PyPI 安装它::

    pip install click

文档内容
----------------------

这部分文档将指引你浏览所有 Click 的使用方法。

.. toctree::
   :maxdepth: 2

   why
   quickstart
   setuptools
   parameters
   options
   arguments
   commands
   prompts
   documentation
   complex
   advanced
   testing
   utils
   bashcomplete
   exceptions
   python3
   wincmd

API Reference
-------------

如果你想查阅一个特定函数、类或者方法的具体信息，请查阅这部分文档。

.. toctree::
   :maxdepth: 2

   api

Miscellaneous Pages
-------------------

.. toctree::
   :maxdepth: 2

   contrib
   changelog
   upgrading
   license
