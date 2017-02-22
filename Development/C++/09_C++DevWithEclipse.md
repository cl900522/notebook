Linux下使用Eclipse开发C++注意点
===============

1. 如果使用了非C/C++标准库(如boost,mysql),编译前需要在Project->Properties->Tool Settings->GCC C++ Linker->Libraries下添加库名称
![](/images/2017/02/c++lib_config.png)
