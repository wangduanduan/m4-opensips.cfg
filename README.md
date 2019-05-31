
<!-- TOC -->

- [1. 使用M4拆分配置文件](#1-使用m4拆分配置文件)
- [2. 实战 拆分opensips.cfg配置文件](#2-实战-拆分opensipscfg配置文件)
  - [2.1. src目录](#21-src目录)
  - [2.2. config.m4](#22-configm4)
  - [2.3. opensips.m4](#23-opensipsm4)
  - [2.4. 最后的处理](#24-最后的处理)
  - [2.5. 总结](#25-总结)
- [3. 扩展](#3-扩展)
  - [3.1. m4获取shell执行结果获取环境变量](#31-m4获取shell执行结果获取环境变量)
- [4. 参考](#4-参考)

<!-- /TOC -->
# 1. 使用M4拆分配置文件

> 你有没有遇到过一种上千行的配置文件，明明只需要修改几个参数，却陷入到茫茫的代码迷雾中, 迷失自我。只觉得：只在此山中，云深不知处。


**什么是m4?**

> m4 是一个通用的宏处理器，由 Brian Kernighan 和 Dennis Ritchie 设计。m4 是基于 Ritchie 早先为 AP-3 小型机开发的 m3 宏处理器扩展的。 [百度百科 M4](https://baike.baidu.com/item/M4/13348112)

**m4有什么功能？**

m4的功能很多，在此只介绍几个比较常用的。

- 文本替换
- 文件引入
- shell调用
- 字符串处理
- 数学运算
- 条件，循环等等

**如何安装m4**

一般linux系统都自带m4命令，可以输入下面的命令测试一下。

```
m4 --version
```

对于windows系统，建议安装[gow](https://github.com/bmatzelle/gow/releases/download/v0.8.0/Gow-0.8.0.exe)。

Gow工具包非常小，但是包含了几十个常用的linux命令，其中就有m4命令。

# 2. 实战 拆分opensips.cfg配置文件

实战代码都在项目仓库中。

> 实际上，m4命令也是opensips官方强烈推荐的配置文件预处理工具。估计上大家都反应这个配置文件很难维护，最近发布的opensips 3.0中，配置文件的预处理本身就已经集成到opensips的命令中去。参考 [Templating opensips.cfg Files v3.0](https://www.opensips.org/Documentation/Templating-Config-Files-3-0)

opensips.cfg配置文件包括三个部分

1. 全局配置
2. 模块加载
3. 路由设置

现在，我们把一个opensips.cfg拆分成3个文件，其实如果某一部分特别大，也是可以拆分成更小的部分。

项目结构如下：

```
- src
    - global.m4
    - module.m4
    - route.m4
- config.m4
- opensips.m4
```
## 2.1. src目录

src目录中分为三个文件，global.m4, module.m4, route.m4。内容基本上都是从原始文件中复制进来的。

着重说明一下global.m4， 这个文件中最后有两个特殊的字符串`CF_LISTEN_IP`,`CF_LISTEN_PORT`在不同的环境中，监听的IP和端口都可能不一样，所以需要提取出来作为宏。

```
listen=udp:CF_LISTEN_IP:CF_LISTEN_PORT   # CUSTOMIZE ME
```

## 2.2. config.m4

config.m4用来定义宏，例如`CF_LISTEN_IP`它的值是`192.168.1.3`，m4在处理文件的时候，遇到CF_LISTEN_IP就会把它替换成192.168.1.3。

```
divert(-1)
define(`CF_LISTEN_IP', `192.168.1.3')
define(`CF_LISTEN_PORT', `8080')
divert(0)dnl

```

注意点：
- 开头的divert(-1)，是必须的，不理解也没关系
- 结尾的divert(0)dnl，也是必须的，不理解也没关系
- 最后，config.m4文件的最后要有一个空行，这个是必须的

## 2.3. opensips.m4 

这个文件用来引入其他分散的文件

```
include(`./src/global.m4')
include(`./src/module.m4')
include(`./src/route.m4')

```

## 2.4. 最后的处理

```sh
m4 config.m4 opensips.m4 > opensips.cfg
```

最终产生的opensips.cfg就是真正可用的配置文件。可以使用opensips自带的命令去检查生成的配置文件是否有效。

```sh
[INSTALL_PATH]/sbin/opensips -C opensips.cfg
```

## 2.5. 总结

- 将文件分散，避免配置文件过大，导致难以维护
- 将经常需要改动的参数单独拿出来作为一个单独的配置文件，方便维护

# 3. 扩展

## 3.1. m4获取shell执行结果获取环境变量

如下，

```
# test-shell.m4
define(`user', esyscmd(`printenv USER'))
user

```

# 4. 参考

- https://www.gnu.org/savannah-checkouts/gnu/m4/manual/m4-1.4.18/m4.html
- https://baike.baidu.com/item/M4/13348112
- https://stackoverflow.com/questions/2477650/unix-get-environment-variable
- https://stackoverflow.com/questions/5346119/in-m4-how-do-you-include-a-file-that-has-an-environment-variable-in-its-name
- The Unknown Power Tool: m4, Part Two  http://www.jpeek.com/articles/linuxmag/2005-03/
- https://github.com/slopjong/automake-example/issues/2
- https://www.kancloud.cn/digest/gun-m4/99015
- 让这世界再多一份 GNU m4 教程 https://www.kancloud.cn/digest/gun-m4/99011
- https://www.opensips.org/Documentation/Tools
- https://www.opensips.org/Documentation/Templating-Config-Files-3-0

