# Anti-GFW Websocks

在github上发现了一个可以用来搭梯子的项目:[vproxy](https://github.com/wkgcass/vproxy)。  
这个项目虽然是用来做负载均衡和服务发现之类的，但是也提供了socks5支持，甚至还做了类似ss的`(本地socks5 agent)+server`模式的应用，搭梯子非常方便。

本文（包括原作者的vproxy项目）纯属技术交流，请勿用于非法用途。

## 文档

参考了原仓库的如下文档：

* [The WebSocks Protocol](https://github.com/wkgcass/vproxy/blob/master/doc/websocks.md)：agent和server之间通信使用的协议细节。
* [扩展应用](https://github.com/wkgcass/vproxy/blob/master/doc_zh/extended-app.md)：具体如何启动agent和server。

我在这里归纳总结了详细的使用步骤、配置项细节，你可以按步骤做部署和测试：

1. [server部署]()
2. [agent部署]()
3. [Chrome的使用方式]()
4. [MacOS的使用方式]()
5. [iOS的使用方式]()

这里没有列出Android的使用方式，因为Android条条框框比较少，只要支持socks5接入的app，配置了地址后都可以正常使用，具体原理可以看iOS的使用方式那一段。

相关下载：

* [vproxy release页面](https://github.com/wkgcass/vproxy/releases)
* [jre 8 下载](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
