---
title: 在macOS下使用VSCode搭建Go开发环境
date: 2016-09-27 23:35:39
tags: [IT]
categories: Go
---


## 准备
- 安装Go，可以在[golang.org](http://golang.org/)下载（下载最新的），在Terminal中键入`go`可以看到所有支持的commond，这意味着你的Go已经装好了

	**注意事项**：Go安装完毕后一定要配置好`GOPATH`这个环境变量，不然你的VSCode将无法成功部署。
- 去微软的[VSCode官网]()下载VSCode并安装

## vscode-go插件及常用扩展依赖工具

- 安装vscode-go

	在VSCode中cmd+shift+P，输入框中输入`ext install`，回车，在「扩展」面板输入框中输入Go，待服务器返回最新的地址后，点击安装，然后重启，此时你的vscode-go已经安装完毕。
	
- 常用工具

	vscode-go 插件需要一些额外的工具，安装它们你只需要复制到Terminal中执行即可，它们会安装到你的`GOPATH`目录下。

```
	#gocode: 
	go get -u -v github.com/nsf/gocode
	#godef: 
	go get -u -v github.com/rogpeppe/godef
	#golint: 
	go get -u -v github.com/golang/lint/golint
	#go-outline: 
	go get -u -v github.com/lukehoban/go-outline
	#goreturns: 
	go get -u -v sourcegraph.com/sqs/goreturns
	#gorename: 
	go get -u -v golang.org/x/tools/cmd/gorename
	#gopkgs: 
	go get -u -v github.com/tpng/gopkgs
	#go-symbols: 
	go get -u -v github.com/newhook/go-symbols
	#guru: 
	go get -u -v golang.org/x/tools/cmd/guru
```

注：一条一条执行很累人的，你可以简单写进一个脚本，执行脚本。网络条件良好的情况下，去抽根烟回来就好了。

## 使VSCode支持Go语言调试

1. 安装调试工具delve，打开Termina输入如下命令。

	```
	#delve项目中安装文档表示如果你安装了homebrew可以通过 "brew install go-delve/delve/delve" 来安装（我失败了）。
	go get -v -u github.com/peterh/liner github.com/derekparker/delve/cmd/dlv
	```

2. 由于调试工具没有代码签名，此时在Mac下是无法正常使用VSCode调试Go程序的，可以通过如下几个步骤解决问题。

	- 打开`keychain`钥匙串访问
	- 打开菜单，选择「证书助理」->「创建证书」
	- 输入一个名字，比如dlv-cert，然后将**identify type**（身份类型）设置为**self signed root**（自签名根证书），将**certificate type**（证书类型）设置为**code signing**（代码签名），然后选择**let me override defaults**（让我覆盖这些默认值）
	- 点击continue，此处你可以调整一下证书的过期时间，如将默认的365改为3650
	- 继续点击continue直到你看到**specify a location for the certificate**（指定该证书用于的位置），选择**system**
	如果不能就选择login，然后导出到system的keychain中
	- 在keychains里选择system，然后就可以找到新建的证书，右键点击证书，选择**get info**（显示简介），打开trust选项，然后将code signing设置为**always trust**
	- （**Yosemite**）在keychain里选择keys，然后找到证书，右键点击，选择get info，然后选择access control标签，选择allow all applications to access this item，然后保存设置
	- 退出keychain，杀死`taskgated`进程重启该服务，或者直接重启电脑
	- 重新编译出带有代码签名的dlv执行程序。打开Termina进入`$GOPATH/src/github.com/derekparker/delve`，如果你的Go版本为1.5你必须先执行`GO15VENDOREXPERIMENT=1`命令，接下来输入`CERT=dlv-cert make install`命令，这样就可以重新编译出一个带有代码签名的dlv。



>参考文档
	
>- https://github.com/Microsoft/vscode-go
>- https://github.com/derekparker/delve/blob/master/Documentation/installation/osx/install.md


