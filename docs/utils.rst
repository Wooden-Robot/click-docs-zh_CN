实用工具
=========

.. currentmodule:: click

除了 Click 提供的与参数解析和处理接口的功能之外，它还提供了一些附加功能，对编写命令行实用程序非常有用。

打印标准输出
------------------

最明显的帮助是 :func:`echo` 函数，它在很多方面像 Python 的 ``print`` 句或函数一样工作。主要区别在于它在Python 2和Python 3中的工作原理是相同的，它智能地检测错误配置的输出流，并且永远不会出错 (Python 3除外;更多信息请参阅 :ref:`python3-limitations`)。

举例::

    import click

    click.echo('Hello World!')

最重要的是，它可以打印Unicode和二进制数据，不像在Python 3中的内置函数 ``print`` ，它不能输出任何字节。但是，它会默认发出一个尾随的换行符，这需要通过传递 ``nl=False``::

    click.echo(b'\xe2\x98\x83', nl=False)

最后但并非最不重要的一点，就是 :func:`echo` 使用click的智能内部输出流来支持在Windows控制台上支持unicode输出的stdout和stderr。 这意味着只要你使用的是 `click.echo` 你可以输出unicode字符（对于哪些字符可以显示，在默认字体上有一些限制）。该功能在 Click 6.0中是新功能。

.. versionadded:: 6.0

现在单击模拟Windows上的输出流，以通过单独的API支持到Windows控制台的unicode。欲了解更多信息，请参阅
`wincmd`_ 。

.. versionadded:: 3.0

从Click 3.0开始，您也可以通过传递 ``err=True`` ，以下内容轻松地打印到标准错误::

    click.echo('Hello World!', err=True)


.. _ansi-colors:

ANSI颜色
-----------

.. versionadded:: 2.0

从Click 2.0开始，该 :func:`echo` 函数获得了处理ANSI颜色和样式的额外功能。 在Windows上，此功能仅在安装了 `colorama`_ 时可用。如果安装，则ANSI代码将被智能处理。

这主要意味着：

-   Click 的 :func:`echo` 函数功能，如果流没有连接到终端，会自动剥离ANSI颜色代码。
-   该 :func:`echo` 功能将透明地连接到Windows上的终端，并将ANSI代码翻译成终端API调用。这意味着颜色在Windows上的工作方式与其他操作系统上的相同。

关于 `colorama` 支持的注意事项：将自动检测何时 可用色彩并使用它。难道 *不* 叫 ``colorama.init()``!

要安装 `colorama`, 请运行以下命令::

    $ pip install colorama

对于字符串的样式， :func:`style` 函数可以如下使用::

    import click

    click.echo(click.style('Hello World!', fg='green'))
    click.echo(click.style('Some more text', bg='blue', fg='white'))
    click.echo(click.style('ATTENTION', blink=True, bold=True))

关于 :func:`echo` 函数和 :func:`style` 函数也称为单一功能，可用 :func:`secho` 函数::

    click.secho('Hello World!', fg='green')
    click.secho('Some more text', bg='blue', fg='white')
    click.secho('ATTENTION', blink=True, bold=True)


.. _colorama: https://pypi.python.org/pypi/colorama

分页支持
-------------

在某些情况下，您可能希望在终端上显示长文本，并让用户滚动浏览。这可以通过使用与 `echo_via_pager()` 函数类似的 `echo()` 函数来实现 ，但是总是写入标准输出，如果可能的话，可以使用分页器。

举例：

.. click:example::

    @click.command()
    def less():
        click.echo_via_pager('\n'.join('Line %d' % idx
                                        for idx in range(200)))


屏幕清除
---------------

.. versionadded:: 2.0

要清除终端屏幕，您可以使用 `clear()` 从Click 2.0开始提供的功能。它的名字的确如此：它以平台不可知的方式清除整个可见的屏幕：

::

    import click
    click.clear()


从终端获取字符
--------------------------------

.. versionadded:: 2.0

通常，当从终端读取输入时，您将从标准输入读取。但是，这是缓冲输入，直到线路终止才显示。在某些情况下，您可能不想这样做，而是在写入时读取单个字符。

为此，Click提供了 `getchar()` 函数，从终端缓冲区读取单个字符并将其作为Unicode字符返回的功能。

请注意，即使标准输入是管道，此功能也将始终从终端读取。


举例::

     import click

    click.echo('Continue? [yn] ', nl=False)
    c = click.getchar()
    click.echo()
    if c == 'y':
        click.echo('We will go on')
    elif c == 'n':
        click.echo('Abort!')
    else:
        click.echo('Invalid input :(')

请注意，这读取原始输入，这意味着像箭头键的东西将出现在平台的本机转义格式。翻译的字符只有 `^C` 和 `^D` 其分别转换成键盘中断和异常的文件结束。这样做是因为否则，忘记这一点太容易了，并且创建不能正确退出的脚本。

等待键入信号
---------------------

.. versionadded:: 2.0

有时，暂停直到用户按下键盘上的任何键都是有用的。这在Windows ``cmd.exe`` 上将特别有用，默认情况下会在命令执行结束时关闭窗口，而不是等待。

在点击中，这可以通过该 `pause()` 功能来完成。此功能将打印一个快捷的信息给终端（可以自定义），并等待用户按下一个键。除此之外，如果脚本没有交互运行，它也会成为NOP（无操作指令）。

举例::

    import click
    click.pause()


启动编辑
-----------------

.. versionadded:: 2.0

Click 支持自动启动编辑器 :func:`edit` 。这对询问用户多行输入非常有用。它会自动打开用户定义的编辑器或回退到合理的默认值。如果用户关闭编辑器而不保存，则返回值将是None，否则为输入的文本。

用法示例::

    import click

    def get_commit_message():
        MARKER = '# Everything below is ignored\n'
        message = click.edit('\n\n' + MARKER)
        if message is not None:
            return message.split(MARKER, 1)[0].rstrip('\n')

或者，也可以使用该功能通过特定文件名启动文件的编辑器。在这种情况下，返回值总是为 `None` 。


举例::

    import click
    click.edit(filename='/etc/passwd')


启动应用程序
----------------------

.. versionadded:: 2.0

Click 支持通过启动应用程序 :func:`launch`. 可以用来打开与URL或文件类型关联的默认应用程序。例如，这可用于启动Web浏览器或图片浏览器。除此之外，它还可以启动文件管理器并自动选择提供的文件。

用法示例::

    click.launch('http://click.pocoo.org/')
    click.launch('/my/downloaded/file.txt', locate=True)


打印文件名
------------------

由于文件名可能不是Unicode，格式化它们可能有点棘手。通常，在Python 2中比在3上更容易，因为你可以用这个 ``print`` 函数把字节写到stdout中，但是在Python 3中，你总是需要使用Unicode。

这与点击工作的方式是通过 :func:`format_filename` 它尽最大努力将文件名转换为Unicode，永远不会失败。这使得可以在完整的Unicode字符串的上下文中使用这些文件名。

举例::

    click.echo('Path: %s' % click.format_filename(b'foo.txt'))


标准流
----------------

对于命令行工具来说，可靠地访问输入输出流是非常重要的。Python通常提供通过 ``sys.stdout`` , 和友元方法访问这些流，但不幸的是，在2.x和3.x之间有API差异，特别是关于这些流如何响应Unicode和二进制数据。

正因为如此，click提供了 :func:`get_binary_stream` 和
:func:`get_text_stream` 函数，这些函数可以在不同的Python版本以及各种各样的pf终端配置下产生一致的结果。

最终的结果是，这些函数将总是返回一个功能流对象（除非在Python 3中很奇怪的情况;请参阅
:ref:`python3-limitations`).

举例::

    import click

    stdin_text = click.get_text_stream('stdin')
    stdout_binary = click.get_binary_stream('stdout')

.. versionadded:: 6.0

现在 Click 模拟Windows上的输出流，以通过单独的API支持到Windows控制台的unicode。欲了解更多信息，请参阅
`wincmd`_.


智能文件打开
------------------------

.. versionadded:: 3.0

从Click 3.0开始， :class:`File` 通过 :func:`open_file` 函数公开从文件中打开文件的逻辑。它可以智能地打开标准输入/标准输出以及任何其他文件。

举例::

    import click

    stdout = click.open_file('-', 'w')
    test_file = click.open_file('test.txt', 'w')

如果返回stdin或stdout，则返回值被封装在一个特殊的文件中，上下文管理器将阻止文件的关闭。这使得标准流的处理是透明的，你可以像这样使用它::

    with click.open_file(filename, 'w') as f:
        f.write('Hello World!\n')


查找应用程序文件夹
---------------------------

.. versionadded:: 2.0

Very often, you want to open a configuration file that belongs to your
application.  However, different operating systems store these configuration
files in different locations depending on their standards.  Click provides
a :func:`get_app_dir` function which returns the most appropriate location
for per-user config files for your application depending on the OS.

Example usage::

    import os
    import click
    import ConfigParser

    APP_NAME = 'My Application'

    def read_config():
        cfg = os.path.join(click.get_app_dir(APP_NAME), 'config.ini')
        parser = ConfigParser.RawConfigParser()
        parser.read([cfg])
        rv = {}
        for section in parser.sections():
            for key, value in parser.items(section):
                rv['%s.%s' % (section, key)] = value
        return rv


显示进度条
---------------------

.. versionadded:: 2.0

有时候，你有命令行脚本需要处理大量的数据，但是你想要快速地向用户展示一下这个过程需要多长时间。点击支持通过该 :func:`progressbar` 函数进行简单的进度条渲染。

基本用法非常简单：这个想法是你有一个你想要操作的迭代器。对于迭代中的每个项目，可能需要一些时间来处理。所以说你有这样一个循环::

    for user in all_the_users_to_process:
        modify_the_user(user)

要连接一个自动更新的进度条，只需要将代码更改为::

    import click

    with click.progressbar(all_the_users_to_process) as bar:
        for user in bar:
            modify_the_user(user)

Click 会自动将进度条打印到终端，并计算剩余的时间。剩余时间的计算要求可迭代的长度。如果它没有长度，但你知道长度，你可以明确地提供它::

    with click.progressbar(all_the_users_to_process,
                           length=number_of_users) as bar:
        for user in bar:
            modify_the_user(user)

另一个有用的功能是将标签与将在进度条之前显示的进度条相关联::

    with click.progressbar(all_the_users_to_process,
                           label='Modifying user accounts',
                           length=number_of_users) as bar:
        for user in bar:
            modify_the_user(user)

有时，可能需要迭代外部迭代器，并不规则地前进进度条。为此，您需要指定长度（并且不可迭代），并在上下文返回值上使用update方法，而不是直接在其上进行迭代::

    with click.progressbar(length=total_size,
                           label='Unzipping archive') as bar:
        for archive in zip_file:
            archive.extract()
            bar.update(archive.size)
