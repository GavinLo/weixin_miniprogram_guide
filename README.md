# 微信小程序入坑指南
	系统：MacOS
	工具：Homebrew、dnsmasq、Nginx

## 前言
微信小程序与传统的web开发差别不大，可以说是传统web开发的一个扩展。它有天然的优势，那就是微信强大的生态圈。个人认为，小程序必然会迅速发展和壮大起来。按照惯例，事物的发展都存在历史遗留问题，之前的web开发和公众号开发向小程序的移植必然会存在成本问题。同时小程序的开发也有不少坑。这是我入坑之后的开发小结，希望各位开发小程序的朋友们带来一些帮助。

## 调试才是重点
对于小程序的开发，官方提供了IDE和完整的API文档，API里面的坑也需要开发者熟悉，但是开发更重好的是 —— 调试。所以我重点介绍一下小程序的调试方法。

## 温馨提示
* __wx.request的请求地址，不支持ip地址，不支持端口号。__

## 本地开发环境调试
这个很简单，首先勾选“开发环境不校验请求域名...”这个选项。
![](微信小程序开发工具项目选项.png)
然后通过使用一些webserver进行请求转发和修改hosts文件，可以解决wx.request的限制问题。这样，就可以无缝的在IDE上预览和测试你的程序了。

## 真机调试（兼容本地调试）
因为真机预览的时候对于微信小程序IDE来说，不算是开发环境，所以它有严格的域名校验，并且请求地址必须为Https协议。我们希望在局域网里面做真机调试，所以我们需要一个本地的DNS服务器来进行域名解析（真机可能不允许修改hosts文件）。
	
### 安装Homebrew
* 可以在[Homebrew的官网](https://brew.sh)查看其安装方法
* 也可以使用以下命令安装：

		/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

### 安装dnsmasq
dnsmasq是一个轻量级的DNS服务器环境，非常适用于局域网。

#### 使用brew安装
	brew install dnsmasq
	
#### 安装完成后，可以使用以下命令，查看dnsmasq的启动方法
	brew info dnsmasq
	
#### 启动dnsmasq
	sudo brew services start dnsmasq
#### 在启动的使用，有可能会提示如下信息：
	Error: Running Homebrew as root is extremely dangerous and no longer supported.
	As Homebrew does not drop privileges on installation you would be giving all
	build scripts full access to your system.
#### 这时候可以使用su登录root账户之后，再使用以下命令解决
	brew services start dnsmasq
	
#### 关闭dnsmasq
	sudo brew services stop dnsmasq
	如果出现于启动时同样的错误，可以使用同样的方法解决
	
#### 拷贝配置文件到(/usr/local/etc)，之后可以直接修改这个配置文件
	sudo cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
#### 同时新建上游DNS配置文件(resolv.dnsmasq.conf)
	sudo touch /usr/local/etc/resolv.dnsmasq.conf
	
#### 修改dnsmasq.conf
	# 在配置文件中添加如下设置
	resolv-file=/usr/local/etc/resolv.dnsmasq.conf #设置上游DNS文件
	strict-order #让上游DNS文件的DNS查询顺序严格按照文件中的顺序
	listen-address=127.0.0.1,yyy.yyy.yyy.yyy #这是监听ip，希望仅本机有用的，添加127.0.0.1即可。希望局域网其他机器都能使用的，还需添加后面的yyy地址
	address=/xxx.com/yyy.yyy.yyy.yyy #xxx为自定义局域网中你所希望访问的域名，它会被解释为后面的yyy地址

#### 修改resolv.dnsmasq.conf
	# 格式：nameserver xxx.xxx.xxx.xxx
	nameserver 127.0.0.1 #一般第一行设置为使用本机作为首个DNS，后面可以设置一些其他常用的DNS地址，以便可以正常上网
	nameserver 114.114.114.114
	nameserver 8.8.8.8
	nameserver 8.8.4.4
	
#### 测试
	dig xxx.com
	
#### 删除缓存
	sudo pkill dnsmasq 

### 注册域名
因为小程序在请求发送时有严格的校验，需要SSL证书进行认证。所以需要注册一个域名。
注册域名后，无论域名在哪里注册，最好使用腾讯云提供的SSL证书，具体申请流程可以参考[这里](https://www.qcloud.com/document/product/400/6814)。

## 总结
一切就绪后，开启dnsmasq和服务器，就可以愉快的玩耍了。