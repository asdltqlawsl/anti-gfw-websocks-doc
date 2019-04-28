# Anti-GFW using Websocks

在github上发现了一个可以用来搭梯子的项目:[vproxy](https://github.com/wkgcass/vproxy)。  
这个项目虽然是用来做负载均衡和服务发现之类的，但是也提供了socks5支持，甚至还做了类似ss的`(本地socks5 agent)+server`模式的应用，搭梯子非常方便。

本文（包括原作者的vproxy项目）纯属技术交流，请勿用于非法用途。

## 文档

参考了原仓库的如下文档：

* [The WebSocks Protocol](https://github.com/wkgcass/vproxy/blob/master/doc/websocks.md)：agent和server之间通信使用的协议细节。
* [扩展应用](https://github.com/wkgcass/vproxy/blob/master/doc_zh/extended-app.md)：具体如何启动agent和server。

我在这里归纳总结了详细的使用步骤、配置项细节，你可以按步骤做部署和测试：

1. [server部署](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/server%E9%83%A8%E7%BD%B2.md)
2. [agent部署](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/agent%E9%83%A8%E7%BD%B2.md)
3. [Chrome的使用方式](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/Chrome%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F.md)
4. [MacOS的使用方式](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/MacOS%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F.md)
5. [iOS的使用方式](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/iOS%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F.md)
6. [Android的使用方式](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/Android%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F.md)

相关下载：

* [vproxy release页面](https://github.com/wkgcass/vproxy/releases)
* [jdk 11 下载](https://jdk.java.net/11/)
* [agent配置文件模板](https://raw.githubusercontent.com/wkgcass/vproxy/master/src/test/resources/websocks-agent-example.conf)
* [常用pac文件](https://raw.githubusercontent.com/petronny/gfwlist2pac/master/gfwlist.pac)
