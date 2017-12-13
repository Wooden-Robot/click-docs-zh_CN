测试 Click 程序
==========================

.. currentmodule:: click.testing

对于基础的测试，Click 提供了 :mod:`click.testing` 模块，它可以调用命令行程序然后检测它的功能。

这类工具真的只能用于测试，因为为了实现简单它们会更改整个解释器的状态，而且不是线程安全的。

基础测试
-------------

:class:`CliRunner` 是测试 Click 程序最基础的功能，它能够调用命令作为命令行脚本。
:meth:`CliRunner.invoke` 方法可以隔离环境中运行命令行脚本并以字节和二进制格式捕获输出数据。

返回结果是一个 :class:`Result` 对象，它可以捕获输出数据，退出代码，附加可选异常。

例如::

    import click
    from click.testing import CliRunner

    def test_hello_world():
        @click.command()
        @click.argument('name')
        def hello(name):
            click.echo('Hello %s!' % name)

        runner = CliRunner()
        result = runner.invoke(hello, ['Peter'])
        assert result.exit_code == 0
        assert result.output == 'Hello Peter!\n'

对于子命令测试，子命令的名称必须在 :meth:`CliRunner.invoke` 方法 `args` 中的指定。

例如::

    import click
    from click.testing import CliRunner
    
    def test_sync():
        @click.group()
        @click.option('--debug/--no-debug', default=False)
        def cli(debug):
            click.echo('Debug mode is %s' % ('on' if debug else 'off')) 
    
        @cli.command()
        def sync():
            click.echo('Syncing')
    
        runner = CliRunner()
        result = runner.invoke(cli, ['--debug', 'sync'])
        assert result.exit_code == 0
        assert 'Debug mode is on' in result.output
        assert 'Syncing' in result.output

文件系统隔离
---------------------

对于基础的想要使用文件系统的命令行工具。:meth:`CliRunner.isolated_filesystem` 方法将非常有用，它会创建一个空的文件夹，并将当前路径设置为空文件夹的路径。

例如::

    import click
    from click.testing import CliRunner

    def test_cat():
        @click.command()
        @click.argument('f', type=click.File())
        def cat(f):
            click.echo(f.read())

        runner = CliRunner()
        with runner.isolated_filesystem():
            with open('hello.txt', 'w') as f:
                f.write('Hello World!')

            result = runner.invoke(cat, ['hello.txt'])
            assert result.exit_code == 0
            assert result.output == 'Hello World!\n'

输入流
-------------

测试装饰器也可以用来为输入流（stdin）提供输入数据。这在测试提示时会非常有用，比如::

    import click
    from click.testing import CliRunner

    def test_prompts():
        @click.command()
        @click.option('--foo', prompt=True)
        def test(foo):
            click.echo('foo=%s' % foo)

        runner = CliRunner()
        result = runner.invoke(test, input='wau wau\n')
        assert not result.exception
        assert result.output == 'Foo: wau wau\nfoo=wau wau\n'

注意：因为提示是模拟的，所以写入输入数据和输出数据也是模拟的。如果预计会被隐藏的输入那么显然结果不会发生。
