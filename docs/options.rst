.. _options:

选项
=======

.. currentmodule:: click

通过 :func:`option` 装饰器可以给命令增加选项。通过配置参数来控制不同的选项。Click 中的选项不同于 :ref:`位置参数 <arguments>`。

基本的值选项
-------------------

最基本的选项是值选项。这种类型的选项接受一个是值得参数。如果没有指定值的类型，那么将使用默认类型 :data:`STRING`。默认情况下，参数的名称为第一个
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

Dynamic Defaults for Prompts
----------------------------

The ``auto_envvar_prefix`` and ``default_map`` options for the context
allow the program to read option values from the environment or a
configuration file.  However, this overrides the prompting mechanism, so
that the user does not get the option to change the value interactively.

If you want to let the user configure the default value, but still be
prompted if the option isn't specified on the command line, you can do so
by supplying a callable as the default value. For example, to get a default
from the environment:

.. click:example::

    @click.command()
    @click.option('--username', prompt=True,
                  default=lambda: os.environ.get('USER', ''))
    def hello(username):
        print("Hello,", username)

Callbacks and Eager Options
---------------------------

Sometimes, you want a parameter to completely change the execution flow.
For instance, this is the case when you want to have a ``--version``
parameter that prints out the version and then exits the application.

Note: an actual implementation of a ``--version`` parameter that is
reusable is available in Click as :func:`click.version_option`.  The code
here is merely an example of how to implement such a flag.

In such cases, you need two concepts: eager parameters and a callback.  An
eager parameter is a parameter that is handled before others, and a
callback is what executes after the parameter is handled.  The eagerness
is necessary so that an earlier required parameter does not produce an
error message.  For instance, if ``--version`` was not eager and a
parameter ``--foo`` was required and defined before, you would need to
specify it for ``--version`` to work.  For more information, see
:ref:`callback-evaluation-order`.

A callback is a function that is invoked with two parameters: the current
:class:`Context` and the value.  The context provides some useful features
such as quitting the application and gives access to other already
processed parameters.

Here an example for a ``--version`` flag:

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

The `expose_value` parameter prevents the pretty pointless ``version``
parameter from being passed to the callback.  If that was not specified, a
boolean would be passed to the `hello` script.  The `resilient_parsing`
flag is applied to the context if Click wants to parse the command line
without any destructive behavior that would change the execution flow.  In
this case, because we would exit the program, we instead do nothing.

What it looks like:

.. click:run::

    invoke(hello)
    invoke(hello, args=['--version'])

.. admonition:: Callback Signature Changes

    In Click 2.0 the signature for callbacks changed.  For more
    information about these changes see :ref:`upgrade-to-2.0`.

Yes Parameters
--------------

For dangerous operations, it's very useful to be able to ask a user for
confirmation.  This can be done by adding a boolean ``--yes`` flag and
asking for confirmation if the user did not provide it and to fail in a
callback:

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

And what it looks like on the command line:

.. click:run::

    invoke(dropdb, input=['n'])
    invoke(dropdb, args=['--yes'])

Because this combination of parameters is quite common, this can also be
replaced with the :func:`confirmation_option` decorator:

.. click:example::

    @click.command()
    @click.confirmation_option(prompt='Are you sure you want to drop the db?')
    def dropdb():
        click.echo('Dropped all tables!')

.. admonition:: Callback Signature Changes

    In Click 2.0 the signature for callbacks changed.  For more
    information about these changes see :ref:`upgrade-to-2.0`.

Values from Environment Variables
---------------------------------

A very useful feature of Click is the ability to accept parameters from
environment variables in addition to regular parameters.  This allows
tools to be automated much easier.  For instance, you might want to pass
a configuration file with a ``--config`` parameter but also support exporting
a ``TOOL_CONFIG=hello.cfg`` key-value pair for a nicer development
experience.

This is supported by Click in two ways.  One is to automatically build
environment variables which is supported for options only.  To enable this
feature, the ``auto_envvar_prefix`` parameter needs to be passed to the
script that is invoked.  Each command and parameter is then added as an
uppercase underscore-separated variable.  If you have a subcommand
called ``foo`` taking an option called ``bar`` and the prefix is
``MY_TOOL``, then the variable is ``MY_TOOL_FOO_BAR``.

Example usage:

.. click:example::

    @click.command()
    @click.option('--username')
    def greet(username):
        click.echo('Hello %s!' % username)

    if __name__ == '__main__':
        greet(auto_envvar_prefix='GREETER')

And from the command line:

.. click:run::

    invoke(greet, env={'GREETER_USERNAME': 'john'},
           auto_envvar_prefix='GREETER')

The second option is to manually pull values in from specific environment
variables by defining the name of the environment variable on the option.

Example usage:

.. click:example::

    @click.command()
    @click.option('--username', envvar='USERNAME')
    def greet(username):
        click.echo('Hello %s!' % username)

    if __name__ == '__main__':
        greet()

And from the command line:

.. click:run::

    invoke(greet, env={'USERNAME': 'john'})

In that case it can also be a list of different environment variables
where the first one is picked.

Multiple Values from Environment Values
---------------------------------------

As options can accept multiple values, pulling in such values from
environment variables (which are strings) is a bit more complex.  The way
Click solves this is by leaving it up to the type to customize this
behavior.  For both ``multiple`` and ``nargs`` with values other than
``1``, Click will invoke the :meth:`ParamType.split_envvar_value` method to
perform the splitting.

The default implementation for all types is to split on whitespace.  The
exceptions to this rule are the :class:`File` and :class:`Path` types
which both split according to the operating system's path splitting rules.
On Unix systems like Linux and OS X, the splitting happens for those on
every colon (``:``), and for Windows, on every semicolon (``;``).

Example usage:

.. click:example::

    @click.command()
    @click.option('paths', '--path', envvar='PATHS', multiple=True,
                  type=click.Path())
    def perform(paths):
        for path in paths:
            click.echo(path)

    if __name__ == '__main__':
        perform()

And from the command line:

.. click:run::

    import os
    invoke(perform, env={'PATHS': './foo/bar%s./test' % os.path.pathsep})

Other Prefix Characters
-----------------------

Click can deal with alternative prefix characters other than ``-`` for
options.  This is for instance useful if you want to handle slashes as
parameters ``/`` or something similar.  Note that this is strongly
discouraged in general because Click wants developers to stay close to
POSIX semantics.  However in certain situations this can be useful:

.. click:example::

    @click.command()
    @click.option('+w/-w')
    def chmod(w):
        click.echo('writable=%s' % w)

    if __name__ == '__main__':
        chmod()

And from the command line:

.. click:run::

    invoke(chmod, args=['+w'])
    invoke(chmod, args=['-w'])

Note that if you are using ``/`` as prefix character and you want to use a
boolean flag you need to separate it with ``;`` instead of ``/``:

.. click:example::

    @click.command()
    @click.option('/debug;/no-debug')
    def log(debug):
        click.echo('debug=%s' % debug)

    if __name__ == '__main__':
        log()

.. _ranges:

Range Options
-------------

A special mention should go to the :class:`IntRange` type, which works very
similarly to the :data:`INT` type, but restricts the value to fall into a
specific range (inclusive on both edges).  It has two modes:

-   the default mode (non-clamping mode) where a value that falls outside
    of the range will cause an error.
-   an optional clamping mode where a value that falls outside of the
    range will be clamped.  This means that a range of ``0-5`` would
    return ``5`` for the value ``10`` or ``0`` for the value ``-1`` (for
    example).

Example:

.. click:example::

    @click.command()
    @click.option('--count', type=click.IntRange(0, 20, clamp=True))
    @click.option('--digit', type=click.IntRange(0, 10))
    def repeat(count, digit):
        click.echo(str(digit) * count)

    if __name__ == '__main__':
        repeat()

And from the command line:

.. click:run::

    invoke(repeat, args=['--count=1000', '--digit=5'])
    invoke(repeat, args=['--count=1000', '--digit=12'])

If you pass ``None`` for any of the edges, it means that the range is open
at that side.

Callbacks for Validation
------------------------

.. versionchanged:: 2.0

If you want to apply custom validation logic, you can do this in the
parameter callbacks.  These callbacks can both modify values as well as
raise errors if the validation does not work.

In Click 1.0, you can only raise the :exc:`UsageError` but starting with
Click 2.0, you can also raise the :exc:`BadParameter` error, which has the
added advantage that it will automatically format the error message to
also contain the parameter name.

Example:

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

And what it looks like:

.. click:run::

    invoke(roll, args=['--rolls=42'])
    println()
    invoke(roll, args=['--rolls=2d12'])
