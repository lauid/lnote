---
title: kubernetes安装
date: 2017-08-04 17:53:20
tags: kubernetes
---

## 安装minikube

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.7.1/minikube-darwin-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/
```


## 启动minikube
```
minikube start
minikube status
```


## 安装kubectl

```
curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/v1.3.0/bin/darwin/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```


## 查看

```
kubectl get pods --all-namespaces
eval $(minikube docker-env)
```


## 参考doc
[Kubernetes in 30 minutes with minikube on Mac](http://www.rushiagr.com/blog/2016/08/16/kubernetes-in-30-minutes-with-minikube-on-mac/)
