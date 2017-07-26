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


# 10个Android学习网站

1. [Android Developers][f0cc4430]
[f0cc4430]: https://developer.android.com/index.html "Android Developers"
作为一个Android开发者，官网的资料当然不可错过，从设计，培训，指南，文档，都不应该错过，在以后的学习过程中慢慢理解体会。

2. [Android Guides - CodePath](http://guides.codepath.com/android)
CodePath是国外一个技术培训机构，主要培训iOS 和android开发，而CodePath将Android Guides放在Github，已经获得了4000+个赞，对于Android初学这特别适合，而且浅显易懂。

3. [Android tutorial - TutorialSpoint](http://www.tutorialspoint.com/android/)
TutorialSpoint是一个专业的技术教程网站，基本上我们所熟知的热门技术，都能在这里找到教程，知识点覆盖的特别全，而且代码风格也很不错，同时也适合初学着；更人性化的是，所有教程提供离线PDF下载。

4. [Android Development - Vogella](http://www.vogella.com/tutorials/android.html)
Vogella提供的Android开发教程也是可圈可点的，可能知识点覆盖不是特别全，但是单个知识点，Vogella讲解的还是很详细的。

5. [AndroidHive](http://www.androidhive.info/)
AndroidHive是一个个人博客，主要写Android开发的教程，虽然只是一个人，但却提供了绘图，到写教程，功能视频演示，也表现出了博主的专业与敬业，博主写的东西也是跟随新技术，可实用性特别强。

6. [Android SDK - Tuts+ Code](http://code.tutsplus.com/categories/android-sdk)
Tuts+是一个技术教程，课程和电子书的网站，基本上热门的技术都提供了，他的教程主要是免费的，而课程，电子书是有偿的，由于其专业性，大多教程都是高精华的。

7. [Lynda](http://www.lynda.com/)
Lynda是一个在线学习网站，该网站提供技术、设计等很多的课程。

8. [Android Questions - Stack Overflow](http://stackoverflow.com/questions/tagged/android)
Stackoverflow是一个技术在线问答网站，几乎平常遇到的所有技术网站，在这里都能找到答案，而且你提问的问题，上面有很多大牛会很热心回答。

9. [Search · android - Github](https://github.com/search?utf8=%E2%9C%93&q=android)
Github是一个基于Git的代码托管工具，几乎所有知名的开源软件都选择Github来托管，而很多Android开发者也都选择Github，几乎常见的Demo在Github都能找到类似的。

10. [Android Archives | Java Code Geeks](http://www.javacodegeeks.com/tutorials/android-tutorials/android-core-tutorials/)
Java Code Geeks主要是一个Java教程的网站，而它提供的Android教程，一步一步，还有配图，使初学者没有太大压力。
