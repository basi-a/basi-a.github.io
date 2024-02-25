---
title: >-
  linux下运行flutter doctor Android Studio 报错 Unable to find bundled Java version
  version"解决方法
date: 2023-02-16 12:02:49
updated: 2023-02-16 12:02:49
tags: 
  - flutter
  - Linux
  - Android studio
categories: Linux
keywords: "linux下运行flutter doctor Android Studio 报错 Unable to find bundled Java version"
description: "linux下运行flutter doctor Android Studio 报错 Unable to find bundled Java version"
top_img:
comments:
cover:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---

# 问题描述
为了测试自己写的后端代码，想着用`flutter`弄个前端界面打包成apk; 
于是在我安装了`Android Studio`运行`flutter doctor -v`后，报了三个错误;

前两个都是`Android toolchain`的错误，到`android studio`里面, 
找到`File >>Settings >> System Settings >> Android SDK >> SDK Tools >> Android SDK Command-line Tools`
安装，这就解决了第一个错误，之后运行`flutter doctor --android-licenses`，接受全部询问。
到这前两个都解决了。

而第三个在`Android Studio`这里报的`Unable to find bundled Java version.`错误, 废了我好长时间，下面是我咋解决的记录。

# 解决方式
一开始，我查资料不是`windows`的就是`macos`的，就是没找到`linux`的;
但看了一会发现，都说是最新的`Android Studio`没有了`jre`导致的，然后通过复制`jre`解决的;

于是我就找到了我的`jdk`的根目录`/usr/lib/jvm/java-19-openjdk` 用`./bin/jlink --module-path jmods --add-modules java.desktop --output jre` 生成了`jre`
然后创建软连接到`Android Studio`的根目录`/opt/android-studio/`,
而我发现这里的`/opt/android-studio/jbr/`内容和`jre`是一样的；

我觉得应该是最新的`Android Studio`的`jre`不是没有而是改了名字，我就把刚弄好的`jre`软连接删掉了，
在这重新创建软连接，再次运行`flutter doctor -v`没有了错误 
```bash
/opt/android-studio/ $ sudo ln -s jbr jre
/opt/android-studio/ $ env CHROME_EXECUTABLE=edge \
flutter doctor -v
```

至此问题解决了, 俺很欣慰😃
