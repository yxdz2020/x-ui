# x-ui

支持多协议多用户的 xray 面板
原版本由[vaxilu](https://github.com/vaxilu)进行开发，项目地址请参考[链接](https://github.com/vaxilu/x-ui)  
本项目由原项目fork而来,初衷只是为了学习golang，并添加自己觉得有用的功能。  
其中Master分支开发功能，develop分支用于提交Pr  
欢迎大家使用并反馈意见，帮助我们更好的改善~  
具体使用教程可以参考个人博客文章[链接](https://coderfan.net/how-to-use-x-ui-pannel-to-set-up-proxies-for-bypassing-gfw.html)
# 变更记录  
2022.06.19：增加Shadowsocs2022新的Cipher，增加节点搜索、一键清除流量功能  
2022.05.14：增加Telegram bot Command控制功能，支持关闭/开启/删除节点等  
2022.04.25：增加SSH登录提醒  
2022.04.23：增加更多Telegram bot提醒功能  
2022.04.16：增加面板设置Telegram bot功能  
2022.04.12：优化Telegram Bot通知提醒  
2022.04.06：优化安装/更新流程，添加Telegram bot机器人推送功能

# 功能介绍

- 系统状态监控
- 支持多用户多协议，网页可视化操作
- 支持的协议：vmess、vless、trojan、shadowsocks、dokodemo-door、socks、http
- 支持配置更多传输配置
- 流量统计，限制流量，限制到期时间
- 可自定义 xray 配置模板
- 支持 https 访问面板（自备域名 + ssl 证书）
- 支持一键SSL证书申请且自动续签
- 增加Telegram bot通知、控制功能
- 更多高级配置项，详见面板
# 安装&升级

```
bash <(curl -Ls https://raw.githubusercontents.com/yxdz2020/x-ui/master/install.sh)
```  
如果你的系统版本比较老旧，安装后报错：GLIBC_2.28 not found，请使用如下命令安装0.3.3.9版本
```  
bash <(curl -Ls https://raw.githubusercontents.com/yxdz2020/x-ui/master/install.sh) 0.3.3.9  
```  
但该版本会在切换xray内核时报错，建议尽快升级系统    


## 手动安装&升级

1. 首先从 https://github.com/vaxilu/x-ui/releases 下载最新的压缩包，一般选择 `amd64`架构
2. 然后将这个压缩包上传到服务器的 `/root/`目录下，并使用 `root`用户登录服务器

> 如果你的服务器 cpu 架构不是 `amd64`，自行将命令中的 `amd64`替换为其他架构

```
cd /root/
rm x-ui/ /usr/local/x-ui/ /usr/bin/x-ui -rf
tar zxvf x-ui-linux-amd64.tar.gz
chmod +x x-ui/x-ui x-ui/bin/xray-linux-* x-ui/x-ui.sh
cp x-ui/x-ui.sh /usr/bin/x-ui
cp -f x-ui/x-ui.service /etc/systemd/system/
mv x-ui/ /usr/local/
systemctl daemon-reload
systemctl enable x-ui
systemctl restart x-ui
```

## 使用docker安装

> 此 docker 教程与 docker 镜像由[Chasing66](https://github.com/Chasing66)提供

1. 安装docker

```shell
curl -fsSL https://get.docker.com | sh
```

2. 安装x-ui

```shell
mkdir x-ui && cd x-ui
docker run -itd --network=host \
    -v $PWD/db/:/etc/x-ui/ \
    -v $PWD/cert/:/root/cert/ \
    --name x-ui --restart=unless-stopped \
    enwaiax/x-ui:latest
```

> Build 自己的镜像

```shell
docker build -t x-ui .
```

## SSL证书申请

> 此功能与教程由[FranzKafkaYu](https://github.com/FranzKafkaYu)提供

脚本内置SSL证书申请功能，使用该脚本申请证书，需满足以下条件:

- 知晓Cloudflare 注册邮箱
- 知晓Cloudflare Global API Key
- 域名已通过cloudflare进行解析到当前服务器

获取Cloudflare Global API Key的方法:
    ![](media/bda84fbc2ede834deaba1c173a932223.png)
    ![](media/d13ffd6a73f938d1037d0708e31433bf.png)

使用时只需输入 `域名`, `邮箱`, `API KEY`即可，示意图如下：
        ![](media/2022-04-04_141259.png)

注意事项:

- 该脚本使用DNS API进行证书申请
- 默认使用Let'sEncrypt作为CA方
- 证书安装目录为/root/cert目录
- 受限于Cloudflare策略，不支持免费域名  
- 输入域名时需为二级域名，不可带`www`
- 本脚本申请证书均为泛域名证书

## Telegram bot使用

> 此功能与教程由[FranzKafkaYu](https://github.com/FranzKafkaYu)提供

X-UI支持通过Telegram bot实现每日流量通知，面板登录提醒以及cmd控制等功能，使用Telegram bot，需要自行申请  
具体申请教程可以参考[博客链接](https://coderfan.net/how-to-use-telegram-bot-to-alarm-you-when-someone-login-into-your-vps.html)  
使用说明:在面板后台设置机器人相关参数，具体包括

- Tg机器人Token
- Tg机器人ChatId
- Tg机器人周期运行时间，可使用crontab语法  

参考语法：
- 30 * * * * * //每一分的第30s进行通知
- @hourly      //每小时通知
- @daily       //每天通知（凌晨零点整）
- @every 8h    //每8小时通知 
- @every 30s   //每30s通知一次

TG通知内容：
- 节点流量使用
- 面板登录提醒
- 节点到期提醒
- 流量预警提醒  
- SSH登录提醒

Command内容：  

- /delete `port`将会删除对应端口的节点
- /restart 将会重启xray服务，该命令不会重启x-ui面板自身
- /status 将会获取当前系统状态
- /enable  `port`将会开启对应端口的节点
- /disable `port`将会关闭对应端口的节点
- /version v1.5.5将会升级xray到1.5.5版本
- /help 获取帮助信息
## 建议系统

- CentOS 7+
- Ubuntu 16+
- Debian 8+

# 常见问题

## 内置SSL证书申请失败
1.请确保你的域名不为免费域名，CF目前暂不支持免费域名通过API进行证书申请  
2.请确保输入的域名为一级域名，不要带www  
3.请勿重复申请，卸载面板本身不会卸载掉证书  
## 安装新面板后登录失效，设置按钮为灰  
1.请清除cookie与浏览器缓存后进行使用  
# Telegram
[CoderfanBaby](https://t.me/CoderfanBaby)  
[FranzKafka‘sPrivateGroup](https://t.me/franzkafayu)


## Stargazers over time

[![Stargazers over time](https://starchart.cc/FranzKafkaYu/x-ui.svg)](https://starchart.cc/FranzKafkaYu/x-ui)
