---
title: ubuntu-install-golang
date: 2020-01-04 09:39:50
tags: golang
---


ubuntu 本身就支持 golang
命令行安装

> sudo apt-get install golang-go
配置运行环境

打开bashrc：
sudo vim ~/.zshrc
添加
GOPATH是go get指令默认下载和安装包的位置，通过go get指令，获取go的包，默认下载到GOPATH所指示的目录中.
GOROOT是go安装的位置，也是go可执行文件的位置，也就是说，当我们命令行中打出go的指令时，系统能不能准确调用go的可执行文件

```
export GOROOT=/usr/local/go   (根据自己的安装路径填写)
export GOPATH=/home/taoyx/program_develop/go_demo(根据自己的想要放置的路径填写)
export PATH=$PATH:$GOPATH:/usr/local/go/bin
````

是修改的配置文件生效

```
source ~/.zshrc
```
安装完成后 用一下命令查看


查看版本： go version
查看go的运行环境： go env

```
➜  ~ go env
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/laugh/.cache/go-build"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/home/laugh/go"
GORACE=""
GOROOT="/usr/lib/go-1.10"
GOTMPDIR=""
GOTOOLDIR="/usr/lib/go-1.10/pkg/tool/linux_amd64"
GCCGO="gccgo"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build161422718=/tmp/go-build -gno-record-gcc-switches"

```