# agent 部署

## 1. 下载二进制文件

参考[release页面](https://github.com/wkgcass/vproxy/releases)选择最新的libs包下载。

```
wget https://github.com/wkgcass/vproxy/releases/download/1.0.0-ALPHA-2/libs.tar.gz
tar zxf libs.tar.gz
cd libs/
```

如果你正在使用linux，那么无需安装jre，可以直接使用打包好的二进制文件启动：

```
## linux:
./vproxy-1.0.0-ALPHA-2-WebSocksAgent-linux ...参数...
```

如果你正在使用macos，那么libs中提供了一个mac app，双击即可运行（不带参数），并且会自动帮你配置好socks5代理。  
当然，你也可以自行执行mac下打包的可执行文件：

```
## macos:
## 双击 vproxy-websocks-agent.app 运行并自动配置
## 或者手动启动:
./vproxy-1.0.0-ALPHA-2-WebSocksAgent-macos ...参数...
```

**注意**：使用二进制文件启动时，当前目录中必须有`libsunec.so`(linux)或者`libsunec.dylib`(macos)。这两个文件可以在[这里](https://github.com/wkgcass/vproxy/tree/master/dep)找到，直接下载即可。在macos app中，该动态连接库文件已经放置好了，可以正常启动。

---

如果你正在使用windows，那么只能使用jre启动jar包了。

```
## windows:
java -D+A:AppClass=WebSocksProxyAgent -jar vproxy-1.0.0-ALPHA-2.jar ...参数...
```

启动参数只有一个，表示配置文件路径。也可以不指定，应用会自动使用`~/vproxy-websocks-agent.conf`作为配置文件（`~/`表示的是"home"路径）。

## 2. 证书配置

如果server端使用的是标准CA签发的证书（比如let's encrypt），那么这一步可以省略，直接使用jre中自带的`cacerts`文件。这个文件在任何jre中都可以找到，但是内容不一定完全一致。  
如果没有jre并且你的环境支持上述的二进制文件，你也可以直接使用[这个](https://github.com/wkgcass/vproxy/raw/master/dep/cacerts)文件，这是从官方graalvm-rc-12的jre里拷贝出来的文件。

如果server端使用的是自签名证书，那么这里还需要增加一步：将自签名证书打到keystore文件里。

```
keytool -importcert -file 证书文件 -alias 随便设置一个名字 -keystore 你的keystore文件路径 -storepass 你的keystore文件密码
```

这一步网速资料比较多，此处给出一个可行的命令，如果有一些自定义需求可以自行搜索资料，这里不再展开赘述了。

## 3. 配置文件

在[这里](https://raw.githubusercontent.com/wkgcass/vproxy/master/src/test/resources/websocks-agent-example.conf)可以找到配置文件模板。

详细说明可以看配置文件模板，这里说明一下关键项的含义，后面会说一下最佳实践：

* `agent.listen` 监听端口，因为是socks5，所以建议设置为1080
* `agent.cacerts.path`和`agent.cacerts.pswd` 这里特别注意下，对于使用jre+jar包启动的agent来说，这两个配置是不需要的，但是如果你使用二进制文件启动，这两个必须设置。path对应了上一步证书配置时的keystore路径，pswd表示密码，jre中自带的cacerts密码为`changeit`
* `agent.gateway` 表示启动网关模式。如果开启，那么这个agent可以被任何“能够访问你的计算机的人”访问到（否则只有本机能访问）。如果你需要让其他机器使用代理功能（比如手机平板，或者其他人的设备），那么这一项需要设置为`on`，否则为了安全考虑，设置为`off`
* `agent.gateway.pac.address` 表示对外暴露一个pac接口。如果开启，那么agent会额外监听一个端口来响应pac文件的请求。这个pac文件会告诉别人，应该使用agent来代理连接。对于iOS设备，这个pac文件是必要的。此外，如果这一项开启，那么上面的`agent.gateway`也要开启，否则别人没办法访问到这个agent。
* `proxy.domain.list` 允许配置域名后缀、正则表达式、pac文件等规则。这个[常用的pac文件](https://raw.githubusercontent.com/petronny/gfwlist2pac/master/gfwlist.pac)是可以正常使用的。也就是说，你可以放心的让所有流量都经过agent，然后agent会自动处理哪些需要代理哪些不需要。如果你愿意，你可以根据模板中的注释配置你需要过滤的域名/ip。如果你需要将任何经过agent的流量都送到server端做代理，那么你可以只配置一项：`/.*/`，表示匹配任何字符，这在chrome+switchyomega的情况下非常有用。

注：`proxy.domain.list`中的pac文件，如果返回值为`"DIRECT"`，那么理解为直连，否则不管返回什么，都认为是“需要代理”。换句话说，agent自己就是代理发起方，它没必要关心pac文件返回的代理地址，而是会使用自己的配置文件中配置的地址。

## 4. 启动

其实启动命令在第一步里已经说了，参照即可。

要说明的是，如果是macos并且你选择使用app来启动程序，那么将读取家目录下的配置文件，并且日志会写到家目录下。

## 5. 停止

对于linux,windows,或者不使用app的macos，停止很简单，`ctrl-c`即可。对于使用app的mac，需要打开`Terminal`，然后输入

```
pkill 'vproxy\-.*\-WebSocksAgent-macos'
```

当进程退出后，socks5配置会自动取消。如果你多次执行app，那么老的进程会停止，新的进程会启动。

## 结语

以上为agent的部署方法。接下来，可以参考各平台的使用方法来实际将agent用起来。

在这里还得提醒：技术本身无罪，请勿非法翻墙。
