# Eclipse创建SharedLibrary
1. 新建C++空项目

2. **在Project->Properties->C/C++Build->Settings->Tool Settings->GCC C++ Compiler->Miscellaneous**
勾选Position Independent Code
![](/images/2017/02/sharedlib_config.png)
3.在Project->Properties->C/C++Build->Settings->Build Artifact
选择Artifact Type为Shared Library
