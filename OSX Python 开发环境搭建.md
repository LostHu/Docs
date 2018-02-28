###OSX Python 开发环境搭建

@(马克飞象)


####需求
>OSX系统自带的Python版本固定，想再升级或者使用不同版本非常不方便。通常系统自带的是2.7版本，但现在很多应用和教程已经是基于3.5+版本了。
>经过多番摸索，找到一种方便的搭建虚拟环境方法Virtualenv。

####包管理工具
Python有很多优秀的组件包，我们开发中经常会使用到这些优秀的官方或者第三方的组件包，而我们需要如何安装管理这些Python组件包呢？
1. easy_install
>easy_install是由PEAK(Python Enterprise Application Kit)开发的setuptools包里带的一个命令，所以使用easy_install实际上是在调用setuptools来完成安装模块的工作。
>

>[安装方法详细说明](https://pypi.python.org/pypi/setuptools#unix-including-mac-os-x-curl)
>```powershell
>curl https://bootstrap.pypa.io/ez_setup.py -o - | python
>```
>**使用示例**
>```powershell
> // 安装
>easy_install Twisted
>// 升级
>easy_install -U Twisted
>// 移除
>easy_install -m Twisted
>```

2. pip
>pip也是一个Python包管理工具，主要是用于安装 PyPI 上的软件包，可以替代 easy_install 工具。
>**pip是 easy_install 的改进版**，提供更好的提示信息，删除package等功能。老版本的python中只有easy_install，没有pip。
>

>**通过easy_install安装**
>```powershell
>sudo easy_install pip
>```
>**使用示例**
>```powershell
> // 安装
>pip install Twisted
>// 升级
>pip install --upgrade Twisted
>// 移除
>pip uninstall Twisted
>```

####安装
使用pip安装Virtualenv
```python
pip install Virtualenv
```

####使用
假如我们要创建的环境叫NewPython
```bash
Virtualenv NewPython
```
这个命令会在当前目录下新建NewPython目录，并在其下创建bin/，lib/，include/，接下来，你会发现有不少东西放在了bin/目录下，其中有python的解释器，以及一些脚本以及我们的activate脚本。
接下来继续运行：
```bash
. NewPython/bin/activate
// 没报错的话，命令行显示如下信息
(NewPython) PCName:当前文件夹 username$
```
环境搭建成功，那么在这个虚拟环境可以任意安装你想要的组件：
```bash
pip install Flask
```
在当前虚拟环境下已经安装好了Flask组件。

####Python IDE
采用**SublimeText**来作为开发IDE。
SublimeText支持在编辑环境中直接编译**Command+B**运行，默认情况下会采用系统当中的Python配置来编译运行。
但对于上面的虚拟环境的编译则会有不同，因为采用的组件包和环境变量都和系统不一样，需要手动指定当前的编译环境和参数。
在SublimeText中选择：
**Tools** -> **Build System** -> **New Build System...**
新建一套编译参数环境，输入以下配置：
```json
{
    "cmd": ["python", "-u", "$file"],
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "selector": "source.python",
    "env": {
		   "PYTHONPATH":"/Users/Lost/Work/ing/Python/PlayFlask/venv/bin/python:/Users/Lost/Work/ing/Python/PlayFlask/venv/lib/python2.7/site-packages"
    }
}
```

另存名为MyPythonBuildSystem.sublime-build文件，然后选择**Tools** -> **Build System** -> **MyPythonBuildSystem**，现在试试**Command+B**运行，开始折腾吧。

####坑
>在**SublimeText2**中无法使用**Ctrl+C**中止当前正在运行的Python程序，必须手动选择**Cancel Build**。
在**SublimeText3**中测试**Ctrl+C**可行。