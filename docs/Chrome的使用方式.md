# Chrome 使用方式

## Windows 和 Linux

Chrome有一个非常常用的代理插件，叫做`Switchy Omega`。

1. [安装该插件](https://chrome.google.com/webstore/detail/padekgcemlokbadohgkifijomclgjgif)
2. 配置socks5规则，其中“代理服务器”设置为`127.0.0.1`，端口设置为agent配置文件中的端口。如果是一个远端机器访问agent，那么“代理服务器”需要设置为agent的host ip地址。
3. 其他PAC、白名单等配置，属于Switchy Omega自己的配置项，脱离本文范畴，而且估计你比我更清楚～
4. 但是，推荐所有流量都经过agent，由agent来决定是否进行代理。如果想让Switchy Omega来决定是否代理，建议将agent的代理列表设置为`/.*/`，表示代理所有经过agent的流量。

![](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/pics/switchy-omega.png?raw=true)

## MacOS

MacOS下的Chrome也可以使用上面描述的Windows和Linux的方法，但是MacOS本身提供了全局的socks5代理功能，建议使用本身的功能而非插件提供的。

参考[MacOS的使用方式](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/MacOS%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F.md)
