# server 部署

部署websocks server有两种方法：

1. heroku app
2. 手动部署

## heroku app

heroku是一个paas服务，可以对外提供http和websocket服务。

但是请注意！登录heroku控制台的时候需要翻墙！

### 1. 注册heroku账号

1. 进入[heroku](https://www.heroku.com/)。
2. 在右上角有一个[Sign up](https://signup.heroku.com/)按钮，点击进入。
3. 接下来会有一个表单让你填写账户信息。除了`Email address`必须填写真实信息外，其他可以随便选。
4. 提交注册信息后，可以到你的邮箱里查看邮件，点链接完成注册。密码需要同时有大写、小写、数字和标点字符。
5. 接下来登录heroku控制台

![](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/pics/register-heroku.png?raw=true)

### 2. 配置你的github仓库

heroku支持直接从github部署代码，所以我们先配置一个github仓库。

1. 登录你的github账号
2. 创建一个仓库，名字随意。但是记得设置为**私有**，其他内容全都不要写，只创建一个完全空的仓库
3. 在你的本地使用git clone将vproxy的websocks分支clone下来：`git clone https://github.com/wkgcass/vproxy.git --branch=websocks5`
4. `cd vproxy/`，然后修改Procfile文件：在末尾的`auth`字段后面，填写自己的`用户名:密码`，如果有多个，用`,`隔开
5. 提交：`git add . && git commit -m 'my user password'`
6. 更换remote，使用这个命令：`git remote remove origin && git remote add origin https://github.com/asdltqlawsl/vproxy-websocks.git`  
    其中，最后的那个地址可以从你刚刚创建的仓库的页面里拷贝下来
7. 使用`git remote -v`检查是否正确
8. `git push --set-upstream origin websocks5`
9. 这时你可以在github页面看到你刚刚提交的内容了

### 3. 部署到heroku

1. 在heroku控制台点击`Create new app`按钮
2. App名称可以随便写，例如：`my-proxy-websocks-example`
3. 在后续页面中应该可以看到`Deployment method`，那里选择`GitHub`  
    ![](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/pics/configure-heroku-choose-github.png?raw=true)
4. 点击`Connect to GitHub`，并选择授权
5. 然后你可以看到一个文本框，输入仓库名称并回车，你可以看到搜索到的仓库列表，选择正确的仓库，点右边的`connect`  
    ![](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/pics/configure-heroku-choose-repo.png?raw=true)
6. 这时可以依次点击`Enable Automatic Deploys`和`Deploy Branch`  
    ![](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/pics/configure-heroku-deploy.png?raw=true)
7. 等待一会儿，你应该可以看到部署成功的提示。点下方出现的`View`按钮或者直接地址栏输入你的app域名，应当能够看到如下内容  
    
    ```
    upgrading related headers are invalid
    ```
    
    这是因为普通http请求会直接被拒绝。当然，你可以稍微改改源码，让它返回一个好看的http页面（可以做伪装）

这样，部署就完成了。后续如果你需要修改密码，直接在你的仓库中提交，任何修改都会自动同步到heroku上。

## 手动部署

在任何一台支持java的机器上都可以部署server端。

### 1. 部署server

有两种方式可以部署server：

1. 使用可执行文件
2. 使用jar包

#### 使用可执行文件

vproxy使用graalvm打了linux和macos的可执行文件，如果你的服务器是linux（或者macos），你可以直接使用可执行文件进行部署。参考[release页面](https://github.com/wkgcass/vproxy/releases)选择最新的libs包下载。

```
wget https://github.com/wkgcass/vproxy/releases/download/1.0.0-ALPHA-2/libs.tar.gz
tar zxf libs.tar.gz
cd libs/
./vproxy-1.0.0-ALPHA-2-WebSocksServer-linux listen $port auth $user:$password
```

其中，`$port`指的是监听地址，`$user:$password`是用户名和密码对，多个可以用逗号分割。

#### 使用jar包

你也可以使用jar包部署，命令稍微长一些：

```
wget https://github.com/wkgcass/vproxy/releases/download/1.0.0-ALPHA-2/libs.tar.gz
tar zxf libs.tar.gz
cd libs/
java -D+A:AppClass=WebSocksProxyServer -jar vproxy-1.0.0-ALPHA-2.jar listen $port auth $user:$password
```

其中，`$port`指的是监听地址，`$user:$password`是用户名和密码对，多个可以用逗号分割。

### 2. 建议使用tmux，或者nohup

由于进程运行在前台，如果你退出terminal那么程序将终止运行。

你可以选择使用tmux，并让程序在tmux里运行。也可以使用nohup并将输出重定向到指定文件。这两个是非常常用的方法，资料很多，这里不再赘述。

### 3. 申请或者自制证书

server端原生没有做TLS支持，所以，为了正常翻墙，你需要使用nginx或者haproxy来做tls offload。  
不过，如果只是代理而非翻墙，你也可以不加TLS支持，连接报文会被墙看得一清二楚。

如果你有一个域名，那么建议使用let's encrypt，或者国内外的云平台签发一个免费证书，否则，你也可以自行使用openssl签发一个自签名证书。

如果是自签名证书，那么你在使用agent时，需要将自己做的自签名证书加入到keystore里面，详情可见[agent部署](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/agent%E9%83%A8%E7%BD%B2.md)。

#### 自签名：

```
mkdir cert
cd cert
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
openssl rsa -in key.pem > key.decrypt.pem
cat cert.pem key.decrypte.pem > certkey.pem
```

第三句会让你输入私钥密码、证书的各种信息。其中，证书信息只有`CN/Common Name`是需要注意的，其他都可以随意填写。  
Command Name这里可以填写你的公网ip（如果你有域名，那么则填写域名）。

另外，第三句最后的`-days`是指证书有效时间，可以调长些，比如100年（36500）。

第四句会将私钥解密，会要求你输入刚才设置的密码.

第五句将证书和私钥放到同一个文件里，在后面部署的时候可能会用到。

### 4. 部署TLS offload

这里以haproxy为例，如果你使用nginx或者其他lb都是没问题的：

#### 安装haproxy

```
apt-get install haproxy
## 如果是centos
yum install haproxy
```

#### 调整配置文件

这里先给出一个参考配置，说明可见后文：

```
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    stats socket /var/lib/haproxy/stats

defaults
    log                     global
    option                  dontlognull
    retries                 3
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    maxconn                 3000

frontend websocks
    bind *:443 ssl crt /etc/haproxy/certkey.pem
    default_backend websocks-local

backend websocks-local
    server s1 127.0.0.1:18686
```

* `frontend`指的是负载均衡监听端口，这里配置成443，并且开启ssl，证书路径为`/etc/haproxy/certkey.pem`。这里的证书建议用自己的域名+let's encrypt签发一份。注意，此处的`certkey.pem`内需要包含证书本体、证书链、私钥。
* `backend`指的是后端服务器组，这里只配置了一个本地的18686端口。

#### 启动haproxy

```
service haproxy restart
```

haproxy应当能够正常启动。但是可能有警告提示说dhparam太小，这个警告可以忽略。

## 结语

到此，整个配置应当全部完毕了，你可以在本机使用`curl -k 'https://xxx'`来验证服务端是否正常工作。

在这里还得提醒：技术本身无罪，请勿非法翻墙。
