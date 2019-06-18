# agent 部署

## 1. 下载java和运行文件

下载java11或者更高版本，以及vproxy运行文件（jar包）。

* [java下载](https://jdk.java.net/11/)
* [vproxy](https://github.com/wkgcass/vproxy/releases/download/1.0.0-ALPHA-4/vproxy-1.0.0-ALPHA-4.jar)

下载你操作系统对应的java文件，然后解压下载下来的压缩包，你应该可以看到一个类似于`jdk-11/`的文件夹。

将下载下来的`vproxy-版本号.jar`文件重命名为`vproxy.jar`，方便后续操作。

然后打开终端，cd到解压上述文件的目录中（windows用户可能不太熟悉，可以自行搜索一下cmd以及cd命令）

启动命令：

```
jdk-11/bin/java -Deploy=WebSocksProxyAgent -jar vproxy.jar 配置文件路径
```

启动命令中的配置文件路径也可以不指定，应用会自动使用`~/vproxy-websocks-agent.conf`作为配置文件（其中`~/`表示的是家目录路径，windows下应该表示`C:\Users\用户名\`，或者`C:\Document and Settings\用户名`这个文件夹）。

如果默认配置文件不存在，那么`vproxy`会启动一个交互式命令行界面，引导你生成配置文件并启动。

如果生成的配置文件运行无误，那么你可以不关心后续步骤。如果无法正常运行，请到家目录删除自动生成的配置文件，然后自行编写或者重新生成。

## 2. 证书配置

如果server端使用的是标准CA签发的证书（比如let's encrypt），那么这一步可以省略。

如果你只是想使用功能，不关心连接是否安全，那么这一步也可以省略，并且在配置文件中添加`agent.cert.verify off`。

如果server端使用的是自签名证书，那么这里还需要增加一步：将自签名证书打到keystore文件里。

```
jdk-11/bin/keytool -importcert -file 证书文件 -alias 随便设置一个名字 -keystore 你的keystore文件路径 -storepass 你的keystore文件密码
```

其中的“证书文件”以`-----BEGIN CERTIFICATE-----`开头。

这一步网上资料比较多，此处给出一个可行的命令，如果有一些自定义需求可以自行搜索资料，这里不再展开赘述了。

## 3. 配置文件

在[这里](https://raw.githubusercontent.com/wkgcass/vproxy/master/src/test/resources/websocks-agent-example.conf)可以找到配置文件模板。

详细说明可以看配置文件模板，这里说明一下关键项的含义，后面会说一下最佳实践：

* `agent.listen` 监听端口，因为是socks5，所以建议设置为1080
* `agent.cacerts.path`和`agent.cacerts.pswd` path对应了上一步证书配置时的keystore路径，pswd表示密码。如果服务端使用的是正常CA签发的证书，那么这两个配置项可以省略
* `agent.cert.verify` 表示是否开启整数校验，默认`on`表示开启，可以设置为`off`，这样可以不设置自定义证书链也能正常使用
* `agent.gateway` 表示启动网关模式。如果开启，那么这个agent可以被任何“能够访问你的计算机的人”访问到（否则只有本机能访问）。如果你需要让其他机器使用代理功能（比如手机平板，或者其他人的设备），那么这一项需要设置为`on`，否则为了安全考虑，设置为`off`
* `agent.gateway.pac.address` 表示对外暴露一个pac接口。如果开启，那么agent会额外监听一个端口来响应pac文件的请求。这个pac文件会告诉别人，应该使用agent来代理连接。如果想让iOS设备访问本机进行代理，那么这个pac文件是必要的。此外，如果这一项开启，那么上面的`agent.gateway`也要开启，否则别人没办法访问到这个agent
* `proxy.domain.list` 允许配置域名后缀、正则表达式、pac文件等规则。这个[常用的pac文件](https://raw.githubusercontent.com/petronny/gfwlist2pac/master/gfwlist.pac)是可以正常使用的。也就是说，你可以放心的让所有流量都经过agent，然后agent会自动处理哪些需要代理哪些不需要。如果你愿意，你可以根据模板中的注释配置你需要过滤的域名/ip。如果你需要将任何经过agent的流量都送到server端做代理，那么你可以只配置一项：`/.*/`，表示匹配任何字符，这在chrome+switchyomega的情况下非常有用

注：`proxy.domain.list`中的pac文件，如果返回值为`"DIRECT"`，那么理解为直连，否则不管返回什么，都认为是“需要代理”。换句话说，agent自己就是代理发起方，它没必要关心pac文件返回的代理地址，而是会使用自己的配置文件中配置的地址。

## 4. 启动

其实启动命令在第一步里已经说了，参照即可。

## 5. 停止

停止很简单，在终端里按下`ctrl-c`即可。

## 结语

以上为agent的部署方法。接下来，可以参考各平台的使用方法来实际将agent用起来。

在这里还得提醒：技术本身无罪，请勿非法翻墙。
