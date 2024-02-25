---
title: ns3 netanim 可视化
date: 2022-06-21 12:47:37
updated: 2022-12-15 18:05:37
tags: ns3
categories: 网络
keywords: "ns3 netanim 可视化"
description: "ns3 netanim 可视化"
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
`ns3 版本 3.36.1`

# 编写脚本

```cpp
  	...
#include "ns3/netanim-module.h"
  	...
int main(){
  	...
  	AnimationInterface anim("xxx.xml"); //生成的xml文件的名字
  	Simulator::Run ();
  	Simulator::Destroy ();
  	return 0;
}
```

# 执行脚本

```bash
~/ns3/ns-allinone-3.36.1/ns-3.36.1 $ ./ns3 run scratch/myfirst.cc
```

# 用NetAnim 打开xml文件

```bash
~/ns3/ns-allinone-3.36.1/netanim-3.108 $ ./NetAnim
```

打开NetAnim后打开生成的xml文件，就可以看见自己创建的节点了