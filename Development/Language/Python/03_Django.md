python 3.6
django 1.11
在安装django1.11的时候出现错误：
Reading https://pypi.python.org/simple/pytz/
Downloading https://pypi.python.org/packages/a4/09/c47e57fc9c7062b4e83b075d418800d322caa87ec0ac21e6308bd3a2d519/pytz-2017.2.zip#md5=f89bde8a811c8a1a5bac17eaaa94383c
The read operation timed out
解决办法：用浏览器打开https://pypi.python.org/packages/a4/09/c47e57fc9c7062b4e83b075d418800d322caa87ec0ac21e6308bd3a2d519/pytz-2017.2.zip#md5=f89bde8a811c8a1a5bac17eaaa94383c后会下载一个zip（pytz-2017.2.zip）。
将这个zip解压。打开cmd进入解压后的文件夹，执行：python setup.py install即可。
