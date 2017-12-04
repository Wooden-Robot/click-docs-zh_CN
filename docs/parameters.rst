参数
==========

.. currentmodule:: click

Click 支持两种类型的脚本参数: 选项和参数。
命令行脚本的作者通常在使用哪个脚本时会有一些混淆，所以这里是对这些差异的简要概述。
正如其名称所示，选项是可选的。虽然参数在合理的范围内是可选的，但是它们在选择
的方式上会受到更多的限制。


为了帮助您在选项和参数之间做出决定，建议仅使用参数，例如转到子命令或输入 文件名, / , URLs，
然后让所有其他选项成为选项。

差异
-----------

参数功能略少于选项。以下功能仅适用于选项:

*   选项可自动提示缺少输入
*   选项可作为标志（布尔值或其他）
*   选项值可以从环境变量中拉出来，但参数不能
*   选项能完整记录在帮助页面中，但参数不能（这显而易见，因为参数可能过于具体而不能自动记录）

另一方面，与选项不同，参数可以接受任意数量的参数。选项可以严格地只接受固定数量的参数（默认为1）。

参数类型
---------------

参数可以是不同的类型。类型可以用不同的行为来实现，有些是开箱即用的:

``str`` / :data:`click.STRING`:
    表示unicode字符串的默认参数类型。

``int`` / :data:`click.INT`:
    只接受整数的参数。

``float`` / :data:`click.FLOAT`:
    只接受浮点值的参数。

``bool`` / :data:`click.BOOL`:
    接受布尔值的参数。这是自动使用布尔值的标志。如果字符值是: ``1``, ``yes``, ``y``
    和 ``true`` 转化为 `True` ； ``0``, ``no``, ``n`` ， ``false`` 转化为 `False` 。

:data:`click.UUID`:
    接受UUID值的参数。这不是自动识别，而是表示为 :class:`uuid.UUID`.

.. autoclass:: File
   :noindex:

.. autoclass:: Path
   :noindex:

.. autoclass:: Choice
   :noindex:

.. autoclass:: IntRange
   :noindex:

自定义参数类型可以通过子类实现 :class:`click.ParamType` 。对于简单的情况，也支持传递一个失败的 `ValueError` Python函数，尽管不鼓励这么做。

参数名称
---------------

参数（包括选项和参数）都接受一些参数声明的位置参数。每个带有单个短划线的字符串
都被添加为短参数; 每个字符串都以一个双破折号开始。如果添加一个没有任何破折号的
字符串，它将成为内部参数名称，也被用作变量名称。

如果一个参数没有给出一个没有破折号的名字, 那么通过采用最长的参数并将所有的破折号
转换为下划线来自动生成一个名字。
如果一个有 ``('-f', '--foo-bar')`` 的选项，那么该参数名被设置为 `foo_bar` ， 
如果一个有 ``('-x',)`` 的选项, 那么该参数名被设置为 `x` ，
如果一个有 ``('-f', '--filename', 'dest')`` 的选项，那么该参数名被设置为 `dest` 。

实现自定义类型
-------------------------

要实现一个自定义类型，你需要继承这个 :class:`ParamType` 类
类型可以调用有或没有上下文和参数对象，这就是为什么他们需要能够处理这个问题。

下面的代码实现了一个整数类型，除了普通整数之外，还接受十六进制和八进制数字，
并将它们转换为常规整数：::

    import click

    class BasedIntParamType(click.ParamType):
        name = 'integer'

        def convert(self, value, param, ctx):
            try:
                if value[:2].lower() == '0x':
                    return int(value[2:], 16)
                elif value[:1] == '0':
                    return int(value, 8)
                return int(value, 10)
            except ValueError:
                self.fail('%s is not a valid integer' % value, param, ctx)

    BASED_INT = BasedIntParamType()

如你所见, 一个子类需要实现这个 :meth:`ParamType.convert` 方法，并且可以选择提供这个 :attr:`ParamType.name` 属性。后者可用于文档的目的。

