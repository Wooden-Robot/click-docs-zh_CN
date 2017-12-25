Bash 参数补全
=============

.. versionadded:: 2.0

Click 2.0 中的任何 Click 脚本都支持 Bash 参数补全。在完成这项工作时，会有一些限制，但大多数情况下，它是可以正常工作的。

限制
-----------

只有当脚本正确安装时，才可以使用 Bash 参数补全，而不是通过 ``python`` 命令执行。
如何做到这一点，参见 :ref:`setuptools-integration`。此外，Click 目前只支持 Bash 参数补全。

目前，Bash 参数补全是一个无法修改的内部特性。可能未来的版本会有更多的选择。


它能补全什么
-----------------

一般，Bash 参数补全支持补全子命令和参数。子命令随时都可以被补全出来，而参数只有在提供一个 `-` 符号后才可以进行补全。比如::

    $ repo <TAB><TAB>
    clone    commit   copy     delete   setuser
    $ repo clone -<TAB><TAB>
    --deep     --help     --rev      --shallow  -r

激活
----------

为了激活 Bash 补全功能，你需要通知 Bash 你的脚本可以使用补全和如何使用补全。任何 Click 程序都会自动完成这个工作。
一般情况下这项功能通过一个神奇的环境变量 ``_<PROG_NAME>_COMPLETE`` 来工作，其中 ``<PROG_NAME>`` 是你的以 `_` 分割的大写的可执行程序的名称。

如果你的工具叫做 ``foo-bar``, 那么这个神奇的变量应该叫做 ``_FOO_BAR_COMPLETE``。通过使用 ``source`` 将工具名导出，它将吐出可以被激活的激活脚本。

比如说，为 ``foo-bar`` 脚本开启 Bash 补全功能，你需要将下列命令放入到 ``.bashrc``::

    eval "$(_FOO_BAR_COMPLETE=source foo-bar)"

从此以后，你的脚本将启用 Bash 补全功能。

激活脚本
-----------------

上述的激活示例总是在启动时调用你的应用程序。如果你有很多的此类应用程序这将会大大增加你的 shell 记过时间。另一个选择是你可以将上述生成的内容通过
一个文件发送，这也是 Git 和其他系统正在做的事情。

通过下列方式简单地实现::

    _FOO_BAR_COMPLETE=source foo-bar > foo-bar-complete.sh

然后将下列命令放入你的 bashrc 中取代之前的命令::

    . /path/to/foo-bar-complete.sh
