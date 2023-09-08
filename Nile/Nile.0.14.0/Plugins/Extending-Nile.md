# 通过插件扩展Nile
Nile可以通过插件扩展其CLI和Nile Runtime Environment功能。您可以fork此[示例模板](https://github.com/franalgaba/nile-plugin-example)，并按照提供的说明实现所需的功能。

## 工作原理
此实现利用了[Click](https://click.palletsprojects.com/)的可扩展性功能。使用Click和利用Python[入口点](https://packaging.python.org/en/latest/specifications/entry-points/)，我们可以在Python环境中通过依赖关系来项目本地扩展。Nile上的插件实现会查找特定的Python入口点约束，以将命令添加到CLI或NRE中。

1. 如果插件提供CLI命令，请使用Click。
```
# First, import click dependency
import click

# Decorate the method that will be the command name with `click.command`
@click.command()
# Custom parameters can be added as defined in `click`: https://click.palletsprojects.com/en/7.x/options/
def greet():
    """
    Plugin CLI command that does something.
    """
    # Done! Now implement the custom functionality in the command
    click.echo("Hello Nile!")
```

2. 使用pyproject.toml文件中的Poetry插件功能定义插件入口点:
```
# We need to specify that Click commands are entry points in the group `nile_plugins`
[tool.poetry.plugins."nile_plugins.cli"]
# <command_name> = <package_method_location>
"greet" = "nile_greet.main.greet"
```

3. 可选地为Nile运行时环境指定插件入口点。如果不需要实现Click命令，则可以删除cli入口点:
```
[tool.poetry.plugins."nile_plugins.cli"]
"greet" = "nile_greet.main.greet"

[tool.poetry.plugins."nile_plugins.nre"]
"greet" = "nile_greet.nre.greet"
```

> NOTE
在为NRE指定插件入口点之前，请确保将函数签名的第一个参数设置为类实例，即self。这允许插件函数访问NRE实例。

4. 完成！为了更好地理解通过setuptools的python入口点，请查阅[此文档](https://setuptools.pypa.io/en/latest/userguide/entry_point.html#entry-points-for-plugins)。

如何决定是否使用插件？只需从项目中安装/卸载插件依赖即可 😄。

在nile_plugins组下同时使用cli和nre入口点，可以开发强大且易于集成的插件。