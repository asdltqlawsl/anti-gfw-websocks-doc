# server 部署

部署websocks server有多种方法，建议使用第一种或第二种：

1. heroku app（免费）
2. 自动脚本部署（需要有自己的linux vps）
3. 手动部署

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
    
    这是因为普通http请求会直接被拒绝。当然，你可以稍微改改源码，让它返回一个好看的html页面（可以做伪装）

这样，部署就完成了。后续如果你需要修改密码，直接在你的仓库中提交，任何修改都会自动同步到heroku上。

## 自动脚本部署

自动脚本支持任意linux机器，理论上对操作系统没有特殊要求，只是需要使用root用户操作。

在root下，随意找一个目录（建议直接在家目录`/root`）：

```
tmux
curl https://raw.githubusercontent.com/wkgcass/vproxy/websocks5/start-vproxy-websocks-proxy-server.sh > start.sh
chmod +x start.sh
./start.sh
```

当看到`server started on 443`说明启动完成。

这里，使用tmux是为了方便保持java执行和观察日志。

要从tmux里回到外面，可以按下`ctrl+B`，然后按下`D`。要回到之前的session，可以执行`tmux attach -t 0`，其中`0`是session号。

后续要启动，则直接执行`./start.sh`，要退出使用`ctrl+C`。

## 手动部署

在任何一台支持java11的机器上都可以部署server端。如果你没有自己的域名，并且没有特殊的定制项，那么建议直接使用自动脚本部署。

### 1. 部署server

这里默认你的server端是linux。  
你可以使用jdk+jar包的方式部署，也可以使用docker部署。

(关于证书，可以参考后文中的`申请或者自制证书`一小节)

#### 使用jar包

```
## 下载运行时（用jlink裁剪过）如果你在非x86_64机器，或者非linux机器，请自行选择对应架构的java 11。
wget https://github.com/wkgcass/vproxy/releases/download/1.0.0-ALPHA-4/vproxy-runtime-linux.tar.gz
tar zxf vproxy-runtime-linux.tar.gz

## 下载jar包
wget -O vproxy.jar https://github.com/wkgcass/vproxy/releases/download/1.0.0-ALPHA-4/vproxy-1.0.0-ALPHA-4.jar

## 启动
vproxy-runtime-linux/bin/java \
    -Deploy=WebSocksProxyServer \
    -jar vproxy.jar \
    listen $port auth $user:$password \
    ssl pkcs12 证书私钥的pkcs12文件路径 pkcs12pswd 密码 \
    domain 你使用的域名(可选)
```

其中，`$port`指的是监听地址，`$user:$password`是用户名和密码对，多个可以用逗号分割。如果你不需要使用`SSL/TLS`功能，那么`ssl`那一行可以省略。

#### 使用docker

首先需要保证你的server能够使用docker，并且安装了docker。

获取镜像：

```
docker build --no-cache -t vproxy:latest https://raw.githubusercontent.com/wkgcass/vproxy/master/docker/Dockerfile
```

然后正常使用即可。具体可以参考[这里](https://github.com/wkgcass/vproxy/blob/master/doc_zh/docker-example.md)，启动参数和上面一致。不过记得挂载卷，以及开放端口。

### 2. 建议使用tmux，或者screen

由于进程运行在前台，如果你退出terminal那么程序将终止运行。

你可以选择使用tmux，并让程序在tmux里运行。screen也是如此。这两个是非常常用的方法，资料很多，这里不再赘述。

如果你在使用docker，那么在`docker run`的时候加个`-d`参数即可。

### 3. 申请或者自制证书

这里建议使用TLS加密（需要证书）。不过，如果只是代理而非翻墙，你也可以不加TLS支持，只不过连接报文会被中间人看得一清二楚。

如果你有一个域名，那么建议使用let's encrypt，或者国内外的云平台签发一个免费证书，否则，你也可以自行使用openssl签发一个自签名证书。

如果是自签名证书，那么你在使用agent时，需要将自己做的自签名证书加入到keystore里面，详情可见[agent部署](https://github.com/asdltqlawsl/anti-gfw-websocks-doc/blob/master/docs/agent%E9%83%A8%E7%BD%B2.md)。

#### 自签名：

```
mkdir cert
cd cert
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
openssl rsa -in key.pem > key.decrypt.pem
cat cert.pem key.decrypt.pem > certkey.pem
openssl pkcs12 -export -in certkey.pem -out certkey.p12
```

第三句会让你输入私钥密码、证书的各种信息。其中，证书信息只有`CN/Common Name`是需要注意的，其他都可以随意填写。  
Command Name这里可以填写你的公网ip（如果你有域名，那么则填写域名）。

另外，第三句最后的`-days`是指证书有效时间，可以调长些，比如100年（36500）。

第四句会将私钥解密，会要求你输入刚才设置的密码.

第五句将证书和私钥放到同一个文件里。

第六句将证书私钥放到pkcs12文件中，方便部署使用，文件命名为`certkey.p12`。

## 结语

到此，整个配置应当全部完毕了，你可以在本机使用`curl -k 'https://xxx'`来验证服务端是否正常工作。

在这里还得提醒：技术本身无罪，请勿非法翻墙。
