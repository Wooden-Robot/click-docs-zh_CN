.. _arguments:

参数
=========

.. currentmodule:: click

参数的工作方式与 :ref:`options <options>` 类似，但都是位置的。由于其语法性质，它们也仅支持options功能的子集，Click不会记录参数，希望你手动记录它们以避免跳出帮助页面。

基本参数
---------------

最基本的 option 是一个值的简单字符串参数。如果没有提供option，则使用默认值的类型。如果没有提供默认值，则类型被假定为 :data:`STRING`.

例如:

.. click:example::

    @click.command()
    @click.argument('filename')
    def touch(filename):
        click.echo(filename)

它看起来像:

.. click:run::

    invoke(touch, args=['foo.txt'])

可变参数：
------------------

第二个常见的版本是接受具体（或不受限）参数的可变参数。这可以用``nargs``参数来控制.  如果设置为 ``-1``, 则接受无限数量的参数。

该值然后作为元组（tuple）传递。请注意，只有一个参数可以设置为  ``nargs=-1``, 它将作为最后一个参数.

例如:

.. click:example::

    @click.command()
    @click.argument('src', nargs=-1)
    @click.argument('dst', nargs=1)
    def copy(src, dst):
        for fn in src:
            click.echo('move %s to folder %s' % (fn, dst))

它看起来像:

.. click:run::

    invoke(copy, args=['foo.txt', 'bar.txt', 'my_folder'])

请记住问题不在于如何实现这个应用原因是这个特别的例子中参数被定义为字符串。  文件名不能是字符串!  它们会出现在特定的系统中，但不会应用所有，下章将详细讲到。

.. admonition:: 关于非空变量参​​数的注意事项  

   如果你使用 ``argparse``, 你可能会失去从``nargs`` 到 ``+`` 的支持，至少有一个参数是必需的。

   它由 ``required=True``设置支持.  如果可以避免的话，不应该使用它，因为如果可变参数是空的，我们认为脚本应该降级成为noops。原因是很多时候，通过命令行输入通配符来调用脚本，如果通配符为空，则不会出错。

.. _file-args:

文件参数
--------------

由于所有的例子都已经使用了文件名，因此解释如何正确处理文件十分重要。命令行工具更有趣，如果他们使用文件unix的方式接受``-``作为特殊文件引用stdin /标准输出。 

Click 支持通过 :class:`click.File` 类型为你智能地处理文件. 它还为所有版本的Python正确处理unicode和字节，使得脚本保存十分方便。

例如:

.. click:example::

    @click.command()
    @click.argument('input', type=click.File('rb'))
    @click.argument('output', type=click.File('wb'))
    def inout(input, output):
        while True:
            chunk = input.read(1024)
            if not chunk:
                break
            output.write(chunk)

它呈现的效果:

.. click:run::

    with isolated_filesystem():
        invoke(inout, args=['-', 'hello.txt'], input=['hello'],
               terminate_input=True)
        invoke(inout, args=['hello.txt', '-'])

文件路径参数
-------------------

在前面的例子中，文件立即被打开。但是如果我们只想要文件名呢？原始方法是使用默认的字符串参数类型。但是，请记住Click是基于Unicode的，所以字符串将始终是Unicode值。不幸的是，文件名可以是Unicode或字节，具体取决于正在使用的操作系统。因此，这种类型是不够的。

相反，你应该使用 :class:`Path` 自动处理这种模糊的类型。它不仅会返回字节或Unicode，而且还取决于更有意义的内容，但也可以为你进行一些基本检查，例如存在检查。

例如:

.. click:example::

    @click.command()
    @click.argument('f', type=click.Path(exists=True))
    def touch(f):
        click.echo(click.format_filename(f))

它呈现的效果:

.. click:run::

    with isolated_filesystem():
        with open('hello.txt', 'w') as f:
            f.write('Hello World!\n')
        invoke(touch, args=['hello.txt'])
        println()
        invoke(touch, args=['missing.txt'])


文件安全打开
-------------------

 :class:`FileType` 类型有一个需要处理的问题，那就是决定何时打开一个文件。默认行为是对它友善。这意味着它将打开标准输入/标准输出和立即打开的文件。当一个文件不能被打开时，这将给用户直接的反馈，但是它只会在第一次执行一个IO操作时，通过自动将文件包装在一个特殊的包装器中来打开文件。

这种行为可以通过传递 ``lazy=True`` 或者 ``lazy=False`` 构造函数来强制。如果该文件被强制打开, 它将通过显示一个失败的IO操作 :exc:`FileError`.

由于文件打开的文件通常会立即清空文件，只有在开发人员确信这是预期的行为时才应禁用懒惰模式。

强制懒惰模式也是非常有用的，以避免资源处理混淆。如果一个文件在懒惰模式下打开，它会收到一个
``close_intelligently`` 方法，可以帮助确定文件是否需要关闭。这对于参数是不需要的，但对于手动提示 :func:`prompt` 函数是必要的，
因为你不知道是否打开了一个类似stdout的流（之前已经打开）或需要关闭的实际文件。

从Click 2.0开始，也可以通过传递以原子模式打开文件atomic=True。在原子模式下，所有写入操作都将放入同一文件夹中的单独文件中，完成后，文件将移至原始位置。如果其他用户定期读取的文件被修改，这是非常有用的。

环境变量
---------------------

与options一样，参数也可以从环境变量中获取值。然而，与选项不同的是，这只支持显示命名的环境变量.

用法事例:

.. click:example::

    @click.command()
    @click.argument('src', envvar='SRC', type=click.File('r'))
    def echo(src):
        click.echo(src.read())

命令行显示:

.. click:run::

    with isolated_filesystem():
        with open('hello.txt', 'w') as f:
            f.write('Hello World!')
        invoke(echo, env={'SRC': 'hello.txt'})

在这种情况下，它也可以是第一个选择的不同环境变量的列表。

一般来说，这个功能是不推荐的，因为它可能会导致用户很大的困惑。


Option-Like 参数
---------------------

有时候，你想处理看起来像options的参数。例如，假设你有一个名为的文件-foo.txt。如果以这种方式将此作为参数传递，Click将把它作为一个options。

为了解决这个问题，Click执行任何POSIX样式的命令行脚本，并接受字符串--作为选项和参数的分隔符。在--标记之后，所有其他参数被接受为参数。

用法事例:

.. click:example::

    @click.command()
    @click.argument('files', nargs=-1, type=click.Path())
    def touch(files):
        for filename in files:
            click.echo(filename)

从命令行执行:

.. click:run::

    invoke(touch, ['--', '-foo.txt', 'bar.txt'])
