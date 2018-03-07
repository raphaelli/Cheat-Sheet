访问博客[Python Django Cheat Sheet](https://ns96.com/2018/01/30/python-django-note/) 带目录模式查看！


>最近在考虑，这种基础的教程费时费力，实际上没有多大的作用，改作 `Cheat Sheet` 模式的话，实际上更方便且省时省力。后续的技术记录博客，除解决指定问题外，一般都以笔记标题定位的`Cheat Sheet` 模式。

# 环境虚拟化
使用底层虚拟化工具 `virtualenv ` 。

`virtualenv ` 是一个创建隔绝的Python环境的工具。 `virtualenv `创建一个包含所有必要的可执行文件的文件夹，用来使用Python工程所需的包。

通过pip安装virtualenv：

```
$ pip install virtualenv
```

然后安装 `virtualenvwrapper` ,其提供了一系列命令使得和虚拟环境工作变得愉快许多。

安装（确保 virtualenv 已经安装了）：
```
$ pip install virtualenvwrapper
$ export WORKON_HOME=~/Envs
$ source /usr/local/bin/virtualenvwrapper.sh
```

**注意：**对于Windows，您可以使用 `virtualenvwrapper-win` , 在Windows中，`WORKON_HOME` 默认的路径是 `%USERPROFILE%Envs` 。
```
$ pip install virtualenvwrapper-win
```

指令如下：

- mkvirtualenv zqxt：创建运行环境zqxt
- workon zqxt: 工作在 zqxt 环境 或 从其它环境切换到 zqxt 环境
- deactivate: 退出终端环境

其它的：
- rmvirtualenv ENV：删除运行环境ENV
- mkproject mic：创建mic项目和运行环境mic
- mktmpenv：创建临时运行环境
- lsvirtualenv: 列出可用的运行环境
- lssitepackages: 列出当前环境安装了的包

创建的环境是独立的，互不干扰，无需sudo权限即可使用 pip 来进行包的管理。

**当然同时也可以使用 `Docker` 来部署虚拟化环境**

参考[官方环境文档](https://docs.docker.com/compose/django/)

# Django 文件结构

- `urls.py` 网址入口，关联到对应的views.py中的一个函数（或者generic类），访问网址就对应一个函数。
- `views.py` 处理用户发出的请求，从urls.py中对应过来, 通过渲染templates中的网页可以将显示内容，比如登陆后的用户名，用户请求的数据，输出到网页。
- `models.py` 与数据库操作相关，存入或读取数据时用到这个，当然用不到数据库的时候 你可以不使用。
- `forms.py` 表单，用户在浏览器上输入数据提交，对数据的验证工作以及输入框的生成等工作，当然你也可以不使用。
- `templates` 文件夹 `views.py` 中的函数渲染templates中的Html模板，得到动态内容的网页，当然可以用缓存来提高速度。
- `admin.py` 后台，可以用很少量的代码就拥有一个强大的后台。
- `settings.py` Django 的设置，配置文件，比如 DEBUG 的开关，静态文件的位置等
