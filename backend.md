# 后端篇

本篇讲述如何搭建起树洞的后端服务。在你的树洞的整个生命周期内，你都应当遵循零流量原则，即任何和服务器的直接连接都是不可接受的；任何和CDN的直接连接都是不可接受的；任何对树洞域名的DNS请求都是不可接受的；任何与你关联的网络服务(如学校邮箱)与服务器有信息往来都是不可接受的。切勿以自己和身边人的邮箱信息在自己的树洞注册，即便是为了调试邮件发送功能。推荐使用安全环境下的跳板机进行配置，并在自己的电脑上添加iptables规则以阻止意外的访问。

你需要安装 mysql，redis 和 nginx. 其中，mysql 最好不要选用 MariaDB，因为它的默认安装缺少一些插件。

除了 nginx 外，后端服务程序不需要以 root 权限运行。我们的实践是使用一个不在 sudoers 里的常规用户。

你可以选择 clone [nameless](https://github.com/pkuhollow/nameless.git) 或 [thuhole-go-backend](https://github.com/thuhole/thuhole-go-backend.git) 作为你的后端。

你需要安装 golang. ubuntu 或 debian 的包管理器内的golang版本较旧，不适于使用。你可以去官网下载最新版本，并将其安装在系统内。假定你设置的`GOPATH`环境变量是`~/go`，那么你也应当把`$GOPATH/bin`加入`PATH`环境变量中。

随后你可以按照`nameless`或`thuhole-go-backend`内的提示编译服务程序，并在`$GOPATH/bin`中找到它们，即`treehole-services-api`，`treehole-login-api`和`treehole-fallback`.

为了启动它们，你需要适当编写`config.yml`. 你需要注册一系列网络服务，例如注册邮件的发送服务，图床服务等，并在 mysql 中建好数据库和供后端服务使用的用户:

```SQL
create database master;
create user 'username'@'localhost' identified by 'password';
grant all on master.* to 'username'@'localhost';
```

这创建了一个叫`master`的库和一个只能从本地登录的用户`username`，因此无需担心有人猜中你的密码。需要注意的是，你应当保证数据库的编码是`UTF-8`，否则将导致中文的树洞和评论无法发出。你应当将用户名和密码填入`config.yml`中。

在完成配置后，你可以编写 systemd unit 来管理这三个可执行文件的运行:

```systemd
[Unit]
Description=treehole service
Requires=network.target mysql.service redis.service

[Service]
WorkingDirectory=/path/to/parent/dir/of/config.yml/
ExecStart=/path/to/treehole-services-api
Restart=on-failure
RestartSec=15
User=YOURUSERNAME

[Install]
WantedBy=multi-user.target
```

```systemd
[Unit]
Description=treehole fallback
Requires=network.target

[Service]
WorkingDirectory=/path/to/parent/dir/of/config.yml/
ExecStart=/path/to/treehole-fallback
Restart=on-failure
RestartSec=15
User=YOURUSERNAME

[Install]
WantedBy=multi-user.target
```

而由于需要输入盐，`treehole-login-api`不适于使用 systemd 管理。你可以使用`screen`或`tmux`来启动它。请一定保存好你的盐！在它打印出数千行的日志后，你在终端内输入盐时的回显就会被清理掉。

如果你启动好了服务，可以尝试

```shell
curl http://localhost:8081/contents/post?pid=1
```

如果服务正常启动，你应当能得到一个 json 格式的响应。

下一步是配置 nginx. 这一步没有什么复杂的，参考`nameless`或`thuhole-go-backend`内的提示即可。你可以使用`let's encrypt`提供的 SSL 密钥对。在配置时你应当注意
+ 绝对不能暴露服务器的真实地址。可以使用`cloudflare`一类的 CDN 服务为服务器遮风挡雨，但将 DNS 记录指向真实地址的操作即使在调试时也不能进行。有时 CDN 的缓存会对 api 带来不想要的后果（例如给出过时的 api 响应），此时应当适当设置 CDN 的缓存策略。
+ 阻断(DROP)来自CDN服务商的地址之外的所有连接。CDN 服务商会提供他们的地址，而你要做的是在配置中加入如下行
  ```conf
    set_real_ip_from XXX.XXX.XXX.XXX;
    set_real_ip_from YYY.YYY.YYY.YYY;
    set_real_ip_from ZZZ.ZZZ.ZZZ.ZZZ;
    real_ip_header X-Forwarded-For;
    real_ip_recursive on;
  ```
+ 关掉`server_tokens`
+ 配置完成后，只留下443端口供外界访问，而 sshd 服务的端口应当改到不常用端口，并通过VPS供应商提供的控制台阻塞之。

**祝您好运，愿树洞之神与您常在。**
