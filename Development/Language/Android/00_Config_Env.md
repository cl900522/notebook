开发环境配置
===========

# 代理设置
1. 打开主界面，依次选择\[Tools\]、\[Options\]，弹出\[Android SDK Manager - Settings\]窗口；
2. 在\[Android SDK Manager - Settings\]窗口中，在\[HTTP Proxy Server\]和\[HTTP Proxy Port\]输入框内填入mirrors.neusoft.edu.cn和80，并且选中\[Force https://... sources to be fetched using http://...\]复选框；
3. 设置完成后单击\[Close\]按钮关闭\[Android SDK Manager - Settings\]窗口返回到主界面

# Gradle设置
gradle的repositories设置为maven { url 'http://repo1.maven.org/maven2' }可以加快gradle的依赖载入速度
```text
repositories {
    maven { url 'http://repo1.maven.org/maven2' }
}
```
