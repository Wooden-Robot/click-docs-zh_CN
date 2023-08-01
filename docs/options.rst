.. _options:

选项
=======

.. currentmodule:: click

通过 :func:`option` 装饰器可以给命令增加选项。通过配置参数来控制不同的选项。Click 中的选项不同于 :ref:`位置参数 <arguments>`。

基本的值选项
-------------------

最基本的选项是值选项。这种类型的选项接受一个是值的参数。如果没有指定值的类型，那么将使用默认类型 :data:`STRING`。默认情况下，参数的名称为第一个
长选项，如果没有长选项则为第一个短选项。

.. click:example::

    @click.command()
    @click.option('--n', default=1)
    def dots(n):
        click.echo('.' * n)

在命令行中运行:

.. click:run::

   invoke(dots, args=['--n=2'])

这上述例子中选项值的类型是 :data:`INT`，因为值的默认值为数字。

多个值的选项
-------------------

有时候，你需要有多个值得选项。这种选项只支持固定数量的参数。通过 ``nargs`` 参数来配置。多个值将被放入一个元组（tuple）中。 

.. click:example::

    @click.command()
    @click.option('--pos', nargs=2, type=float)
    def findme(pos):
        click.echo('%s / %s' % pos)

在命令行中运行:

.. click:run::

    invoke(findme, args=['--pos', '2.0', '3.0'])

.. _tuple-type:

使用元组（tuple）代替多个值的选项
-----------------------------

.. versionadded:: 4.0

正如你所看到的，使用 `nargs` 来设置一个每个值都是数字的选项，得到的元组（tuple）中都是一样的数字类型。这可能不是你想要的。
通常情况下，你希望元组中包含不同类型的值。你可以直接使用下列的特殊元组达成目的：

.. click:example::

    @click.command()
    @click.option('--item', type=(unicode, int))
    def putitem(item):
        click.echo('name=%s id=%d' % item)

在命令行中运行:

.. click:run::

    invoke(putitem, args=['--item', 'peter', '1338'])

使用元组的选项，`nargs` 会根据元组的长度自动生成，同时自动使用 :class:`click.Tuple`。以下列子可等价表达上述例子：

.. click:example::

    @click.command()
    @click.option('--item', nargs=2, type=click.Tuple([unicode, int]))
    def putitem(item):
        click.echo('name=%s id=%d' % item)

多个选项
----------------

和 ``nargs`` 类似，有时候可能会需要一个参数传递多次，同时记录每次的值而不是只记录最后一个值。
比如，``git commit -m foo -m bar`` 会记录两行 commit 信息：``foo`` 和 ``bar``。这个功能
可以通过 ``multiple`` 参数实现：

例如:

.. click:example::

    @click.command()
    @click.option('--message', '-m', multiple=True)
    def commit(message):
        click.echo('\n'.join(message))

在命令行中运行:

.. click:run::

    invoke(commit, args=['-m', 'foo', '-m', 'bar'])

计数
--------

在一些非常罕见的情况下，使用重复的选项来计数是很有意思的。例如：

.. click:example::

    @click.command()
    @click.option('-v', '--verbose', count=True)
    def log(verbose):
        click.echo('Verbosity: %s' % verbose)

在命令行中运行:

.. click:run::

    invoke(log, args=['-vvv'])

布尔值标记
-------------

布尔值标记用于开启或关闭选项。可以通过以 ``/`` 分割的两个标记来实现开启或关闭选项。（如果 ``/`` 在一个选项名中，Click 会自动识别其为布尔值标记，隐式默认 ``is_flag=True``）。Click 希望你能提供一个开启和关闭标记然后设置默认值。

例如:

.. click:example::

    import sys

    @click.command()
    @click.option('--shout/--no-shout', default=False)
    def info(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)

在命令行中运行:

.. click:run::

    invoke(info, args=['--shout'])
    invoke(info, args=['--no-shout'])

如果你实在不想要一个关闭标记，你只需要定义开启标记，然后手动设置它为标记。

.. click:example::

    import sys

    @click.command()
    @click.option('--shout', is_flag=True)
    def info(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)

在命令行中运行:

.. click:run::

    invoke(info, args=['--shout'])

提示：如果 ``/`` 已经包含在你的选项名中（比如说如果你使用 Windows 风格的参数 ``/`` 是字符串的前缀），你可以使用 ``;`` 来代替 ``/``。

.. click:example::

    @click.command()
    @click.option('/debug;/no-debug')
    def log(debug):
        click.echo('debug=%s' % debug)

    if __name__ == '__main__':
        log()

.. versionchanged:: 6.0

如果你想定义一个别名作为第二个选项名，你需要开头空格消除格式化字符串时的歧义：

例如:

.. click:example::

    import sys

    @click.command()
    @click.option('--shout/--no-shout', ' /-S', default=False)
    def info(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)

.. click:run::

    invoke(info, args=['--help'])

功能开关
----------------

另一种布尔值标记，同时也是功能开关。通过对多个选项设置同一个参数名，同时设置一个标记来实现。提示通过提供 ``flag_value`` 参数，Click 会自动隐式设置 ``is_flag=True``。

设置一个默认值为 `True` 的默认标记。

.. click:example::

    import sys

    @click.command()
    @click.option('--upper', 'transformation', flag_value='upper',
                  default=True)
    @click.option('--lower', 'transformation', flag_value='lower')
    def info(transformation):
        click.echo(getattr(sys.platform, transformation)())

在命令行中运行:

.. click:run::

    invoke(info, args=['--upper'])
    invoke(info, args=['--lower'])
    invoke(info)

.. _choice-opts:

选择选项
--------------

有时候你想要一个值为某个列表内的某一个的参数。这种情况下，你可以使用 :class:`Choice` 类型。它可以实例化一个包含有效值的列表。

例如:

.. click:example::

    @click.command()
    @click.option('--hash-type', type=click.Choice(['md5', 'sha1']))
    def digest(hash_type):
        click.echo(hash_type)

运行如下:

.. click:run::

    invoke(digest, args=['--hash-type=md5'])
    println()
    invoke(digest, args=['--hash-type=foo'])
    println()
    invoke(digest, args=['--help'])

.. _option-prompting:

提示
---------

有时候，你想通过命令行输入没有提供的参数。通过定义一个 prompt 参数可以实现这个功能。

例如:

.. click:example::

    @click.command()
    @click.option('--name', prompt=True)
    def hello(name):
        click.echo('Hello %s!' % name)

运行如下:

.. click:run::

    invoke(hello, args=['--name=John'])
    invoke(hello, input=['John'])

如果你不喜欢默认的提示信息，你可以自己定义：

.. click:example::

    @click.command()
    @click.option('--name', prompt='Your name please')
    def hello(name):
        click.echo('Hello %s!' % name)

运行如下:

.. click:run::

    invoke(hello, input=['John'])

密码提示
----------------

Click 也支持隐藏输入信息和确认，这在输入密码时非常有用：

.. click:example::

    @click.command()
    @click.option('--password', prompt=True, hide_input=True,
                  confirmation_prompt=True)
    def encrypt(password):
        click.echo('Encrypting password to %s' % password.encode('rot13'))

运行如下:

.. click:run::

    invoke(encrypt, input=['secret', 'secret'])

因为这种情况非常普遍，因此你可以直接用 :func:`password_option` 装饰器取代：

.. click:example::

    @click.command()
    @click.password_option()
    def encrypt(password):
        click.echo('Encrypting password to %s' % password.encode('rot13'))

提示时获取动态的默认值
----------------------------

上下文中的 ``auto_envvar_prefix`` 和 ``default_map`` 选项允许程序从环境变量或者配置文件中读取选项的值。
不过这会覆盖提示机制，你将不能够自主输入选项的值。

如果你想要用户自己设置默认值，同时如果命令行没有获取该选项的值仍然使用提示进行输入，你可以提供一个可供调用的默认值。比如说从环境变量中获取一个默认值：

.. click:example::

    @click.command()
    @click.option('--username', prompt=True,
                  default=lambda: os.environ.get('USER', ''))
    def hello(username):
        print("Hello,", username)

回调选项和优先选项
---------------------------

有时候，你想要一个参数去完整地改变程序运行流程。比如，你想要一个 ``--version`` 参数去打印出程序的版本然后退出。

提示：``--version`` 参数功能真正地实现是依靠 Click 中的 :func:`click.version_option`。下面的代码只是做一个简单的展示。

在下面你例子中，你需要明白两个概念：优先参数和回调。优先参数会比其他参数优先处理，回调是参数被处理后将调用回调函数。在优先需要一个参数时优先运行是很必须要的。
比如，如果 ``--version`` 运行前需要 ``--foo`` 参数，你需要让它优于 ``--version`` 运行。详细信息请查看 :ref:`callback-evaluation-order`。

回调是有当前上下文 :class:`Context` 和值两个参数的函数。上下文提供退出程序和访问其他已经生成的参数的有用功能。

下面是一个 ``--version`` 的简单例子:

.. click:example::

    def print_version(ctx, param, value):
        if not value or ctx.resilient_parsing:
            return
        click.echo('Version 1.0')
        ctx.exit()

    @click.command()
    @click.option('--version', is_flag=True, callback=print_version,
                  expose_value=False, is_eager=True)
    def hello():
        click.echo('Hello World!')

`expose_value` 参数可以避免没有用的 ``version`` 参数传入回调函数中。如果没有设置它，一个布尔值将传入 `hello` 脚本中。
`resilient_parsing` 用于在 Click 想在不影响整个程序运行的前提下解析命令行。这个例子中我将退出程序，什么也不做。

如下所示:

.. click:run::

    invoke(hello)
    invoke(hello, args=['--version'])

.. admonition:: Callback Signature Changes

    In Click 2.0 the signature for callbacks changed.  For more
    information about these changes see :ref:`upgrade-to-2.0`.


Yes 参数
--------------

对于一些危险的操作，询问用户是否继续是一个明智的选择。通过添加一个布尔值 ``--yes`` 标记就可以实现，用户如果不提供它，就会得到提示。

.. click:example::

    def abort_if_false(ctx, param, value):
        if not value:
            ctx.abort()

    @click.command()
    @click.option('--yes', is_flag=True, callback=abort_if_false,
                  expose_value=False,
                  prompt='Are you sure you want to drop the db?')
    def dropdb():
        click.echo('Dropped all tables!')

在命令行中运行:

.. click:run::

    invoke(dropdb, input=['n'])
    invoke(dropdb, args=['--yes'])

因为这样的组合很常见，所以你可以用 :func:`confirmation_option` 装饰器来实现：

.. click:example::

    @click.command()
    @click.confirmation_option(prompt='Are you sure you want to drop the db?')
    def dropdb():
        click.echo('Dropped all tables!')

.. admonition:: Callback Signature Changes

    In Click 2.0 the signature for callbacks changed.  For more
    information about these changes see :ref:`upgrade-to-2.0`.

从环境变量中获取值
---------------------------------

Click 有一个非常有用的特性，除了接收常规的参数外它可以从环境变量中接收参数。这个功能可以让工具更容易自动化。比如，你可能想要通过 ``--config`` 参数获取配置文件
，同时又想支持通过提供 ``TOOL_CONFIG=hello.cfg`` 键值对来获取配置文件。

Click 通过两种方式实现这种需求。一种是去自动创建选项所需的环境变量。开启这个功能需要在脚本运行时使用 ``auto_envvar_prefix`` 参数。每个命令和参数将被添加为以
下划线分割的大写变量。如果你有一个叫做 ``foo`` 的子命令，它有一个叫 ``bar`` 的选项，且有一个叫 ``MY_TOOL`` 的前缀，那么变量名就叫 ``MY_TOOL_FOO_BAR``。

用例:

.. click:example::

    @click.command()
    @click.option('--username')
    def greet(username):
        click.echo('Hello %s!' % username)

    if __name__ == '__main__':
        greet(auto_envvar_prefix='GREETER')

在命令行中运行:

.. click:run::

    invoke(greet, env={'GREETER_USERNAME': 'john'},
           auto_envvar_prefix='GREETER')

另一种是通过在选项中定义环境变量的名字来手工从特定的环境变量中获取值。

用例:

.. click:example::

    @click.command()
    @click.option('--username', envvar='USERNAME')
    def greet(username):
        click.echo('Hello %s!' % username)

    if __name__ == '__main__':
        greet()

在命令行中运行:

.. click:run::

    invoke(greet, env={'USERNAME': 'john'})

在这个例子中，也可以使用列表，列表中的第一个值将被选用。

从环境变量中获取多个值
---------------------------------------

由于选项可以接收多个值，从环境变量中获取多个值（字符串）稍微复杂一些。Click 通过定义 type 
同时 ``multiple`` 和 ``nargs`` 的值需要为 ``1`` 以外的值，Click 会运行 :meth:`ParamType.split_envvar_value` 
方法来进行分隔。

默认情况下所有的 type 都将使用空格来分割。但是 :class:`File` 和 :class:`Path` type 是例外，它们两个都遵守操作系统的路径分割规则。
在 Linux 和 OS X 的 Unix系统上，通过 (``:``) 分割，在 Windows 系统上，通过 (``;``) 分割。

用例:

.. click:example::

    @click.command()
    @click.option('paths', '--path', envvar='PATHS', multiple=True,
                  type=click.Path())
    def perform(paths):
        for path in paths:
            click.echo(path)

    if __name__ == '__main__':
        perform()

在命令行中运行:

.. click:run::

    import os
    invoke(perform, env={'PATHS': './foo/bar%s./test' % os.path.pathsep})

其他前缀参数
-----------------------

Click 能够使用除了 ``-`` 以外进行分割的前缀参数。如果你想处理有斜杠 ``/`` 或其他类似的参数，这个特性将非常有用。
注意一般情况下强烈不推荐使用，因为 Click 想要开发者尽可能地保持 POSIX 语法。但是在一些特定情况下，这个特性是很有用的：

.. click:example::

    @click.command()
    @click.option('+w/-w')
    def chmod(w):
        click.echo('writable=%s' % w)

    if __name__ == '__main__':
        chmod()

在命令行中运行:

.. click:run::

    invoke(chmod, args=['+w'])
    invoke(chmod, args=['-w'])

注意如果你想使用 ``/`` 作为前缀字符，如果你想要使用布尔值标记，你需要使用 ``;`` 分隔符替换 ``/``:

.. click:example::

    @click.command()
    @click.option('/debug;/no-debug')
    def log(debug):
        click.echo('debug=%s' % debug)

    if __name__ == '__main__':
        log()

.. _ranges:

范围选项
-------------

使用 :class:`IntRange` type 可以获得一个特殊的方法，它和 :data:`INT` type 有点像，它的值被限定在一个特定的范围内（包含两端的值）。它有两种模式：

- 默认模式（非强制模式），如果值不在区间范围内将会引发一个错误。
- 强制模式，如果值不在区间范围内，将会强制选取一个区间临近值。也就是说如果区间是 ``0-5``，值为 ``10`` 则选取 ``5``，值为 ``-1`` 则选取 ``0``。

例如:

.. click:example::

    @click.command()
    @click.option('--count', type=click.IntRange(0, 20, clamp=True))
    @click.option('--digit', type=click.IntRange(0, 10))
    def repeat(count, digit):
        click.echo(str(digit) * count)

    if __name__ == '__main__':
        repeat()

在命令行中运行:

.. click:run::

    invoke(repeat, args=['--count=1000', '--digit=5'])
    invoke(repeat, args=['--count=1000', '--digit=12'])

如果区间的一端为 ``None``，这意味着这一端将不限制。

使用回调函数进行验证
------------------------

.. versionchanged:: 2.0

如果你想自定义验证逻辑，你可以在回调参数中做这些事。回调方法中既可以改变值又可以在验证失败时抛出错误。

在 Click 1.0 中，你需要抛出 :exc:`UsageError` 错误，但是从 Click 2.0 开始，你也可以抛出 :exc:`BadParameter` 错误，这个错误增加了一些优点，它会自动
格式化包含参数名的错误信息。

例如:

.. click:example::

    def validate_rolls(ctx, param, value):
        try:
            rolls, dice = map(int, value.split('d', 2))
            return (dice, rolls)
        except ValueError:
            raise click.BadParameter('rolls need to be in format NdM')

    @click.command()
    @click.option('--rolls', callback=validate_rolls, default='1d6')
    def roll(rolls):
        click.echo('Rolling a %d-sided dice %d time(s)' % rolls)

    if __name__ == '__main__':
        roll()

在命令行中运行:

.. click:run::

    invoke(roll, args=['--rolls=42'])
    println()
    invoke(roll, args=['--rolls=2d12'])
