---
layout: post
title:  "Service Mesh"
date:   2018-02-25 10:44:18
categories: service mesh
---


最近被 react 虐得不行，想来点清真的。正好前几个月参加*阿里 RocketMQ 深圳技术讲座*，接触到一个新词：云原生应用 ( Cloud Native Application )。而最近自己也在研究 Service Mesh，索性记录一下自己的进展。

之前大家一直说微服务，首当其冲的微服务框架便是 Spring Cloud，之前一直用 Spring Boot，没有太多用  Spring Cloud，课程倒是看了不少。之所以没有用上它，一方面由于历史原因，大多数项目用的基本都是 Dubbo(Dubbox) RPC 框架。很多项目想尝试，后来又觉得改动太大，用 Spring Boot 已经是很不错的进步了。

Spring Cloud功能上来说是很强大，至少比 Dubbo功能丰富，涵盖了服务发现，服务治理，熔断器，日志跟踪，负载均衡等，但是如果真的要使用，大家是不是还要考虑 REST 性能与 RPC 的比较，以及 Spring Config  的配置管理是否适用现有项目等各种实际问题，总感觉水土不服。

Spring Cloud 和 Dubbo 都属于侵入型微服务框架，所以选型很重要，一旦用了就很难去改变。之前想把 Dubbo 改成 Spring Cloud ，后来发现那个代价不是一般高，服务还不能灰度上线。突然想起之前直接用 Protobuf+Netty 做推送，然后看了Dubbox 的代码，底层是 Netty+Kryo，其实和我们之前做的差不多。

既然要说Service Mesh，铺垫也差不多了。Service mesh 译作 ”服务网格“，作为服务间通信的基础设施层。对比上述框架来说，是一种非侵入性框架或者基础架构。也就意味着微服务的那些东西就不用和业务代码凑在一起，业务不用再去关心该用什么去做服务发现，无论是自己用 zookeeper 实现一个简单服务发现，还是用 etcd 来实现，其实都是在增加业务代码的复杂度。Service Mesh就不再重蹈覆辙，上述能力被下放到基础架构中，就像一个操作系统天生就可以完成网络通讯一样。

Service Mesh有以下特点：

1. 将服务发现，重试/超时,监控，追踪等下放到底层基础架构中。
2. 对应用程序透明，非侵入性
3. 作为应用间的透明网络代理



目前比较流行的 Service Mesh 框架有 Istio 和 Linkerd，都可以和 K8S 进行集成。

下面我们就来实践一下Istio。

首先安装 k8s ，如果觉得比较麻烦就直接安装minikube（推荐），之后开始安装 istio，前方注意和谐上网。

```shell
curl -L https://git.io/getLatestIstio | sh -
```

然后添加到环境变量中

```shell
cd istio-0.x
export PATH=$PWD/bin:$PATH
```

前几天还是0.5,现在已经是0.6，更新还是比较迅速的。

下面就增长开始安装:

```shell
~ kubectl apply -f install/kubernetes/istio.yaml
```

安装成功之后确认一下：

```shell
~ kubectl get pods -n istio-system

NAME                             READY     STATUS    RESTARTS   AGE
istio-ca-97bddb669-b7xrx         1/1       Running   4          11d
istio-ingress-777786fbcd-tvtdf   1/1       Running   8          11d
istio-mixer-c976b8854-m4nqg      3/3       Running   12         11d
istio-pilot-776bb8cb7f-7h95b     2/2       Running   8          11d
```

istio就安装成功了，下面来安装bookinfo官方示例。

```shell
~ kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
```

确认一下:

```shell
~ kubectl get pods

NAME                             READY     STATUS    RESTARTS   AGE
details-v1-56cf65ddc7-vz4pr      2/2       Running   12         11d
productpage-v1-b64876bc7-gfp9p   2/2       Running   12         11d
ratings-v1-cccf67989-mjb95       2/2       Running   12         11d
reviews-v1-846fd9c945-8xmqg      2/2       Running   12         11d
reviews-v2-74f5547679-bj6zv      2/2       Running   12         11d
reviews-v3-8d889d8d5-lqc27       2/2       Running   12         11d
```



下面获取网关地址，这里不同的安装方式获取地址的方式是不一样的，这里只演示 minikube。

```shell
~  export GATEWAY_URL=$(kubectl get po -l istio=ingress -n istio-system -o 'jsonpath={.items[0].status.hostIP}'):$(kubectl get svc istio-ingress -n istio-system -o 'jsonpath={.spec.ports[0].nodePort}')

~ echo $GATEWAY_URL                                                                                                                                                             
192.168.99.100:32705
```

然后在浏览器中访问 http://192.168.99.100:32705/productpage 就可以看到 BookInfo 主界面。

istio 有 Intelligent Route,下面也来体验一下：

先查看一下路由信息

```shell
 ~ istioctl get routerules -o yaml                                                                                                                                        
 No resources found.
```

导入 v1路由信息

```shell
 ~/Downloads/istio-0.5.0  istioctl create -f samples/bookinfo/kube/route-rule-all-v1.yaml                                                                                          

Created config route-rule/default/productpage-default at revision 62564
Created config route-rule/default/reviews-default at revision 62565
Created config route-rule/default/ratings-default at revision 62566
Created config route-rule/default/details-default at revision 62567

 ~/Downloads/istio-0.5.0  istioctl get routerules -o yaml                                                                                                                          

apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  creationTimestamp: null
  name: details-default
  namespace: default
  resourceVersion: "62567"
spec:
  destination:
    name: details
  precedence: 1
  route:
  - labels:
      version: v1
---
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  creationTimestamp: null
  name: productpage-default
  namespace: default
  resourceVersion: "62564"
spec:
  destination:
    name: productpage
  precedence: 1
  route:
  - labels:
      version: v1
---
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  creationTimestamp: null
  name: ratings-default
  namespace: default
  resourceVersion: "62566"
spec:
  destination:
    name: ratings
  precedence: 1
  route:
  - labels:
      version: v1
---
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  creationTimestamp: null
  name: reviews-default
  namespace: default
  resourceVersion: "62565"
spec:
  destination:
    name: reviews
  precedence: 1
  route:
  - labels:
      version: v1
---
```

网页上显示为Book Reviews

下面我们替换为 v2路由

```Shell
 ~/Downloads/istio-0.5.0  istioctl create -f samples/bookinfo/kube/route-rule-reviews-test-v2.yaml

Created config route-rule/default/reviews-test-v2 at revision 62803
 ~/Downloads/istio-0.5.0  istioctl get routerule reviews-test-v2 -o yaml 

apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  creationTimestamp: null
  name: reviews-test-v2
  namespace: default
  resourceVersion: "62803"
spec:
  destination:
    name: reviews
  match:
    request:
      headers:
        cookie:
          regex: ^(.*?;)?(user=jason)(;.*)?$
  precedence: 2
  route:
  - labels:
      version: v2
---
```

然后登录jason账户，密码随意，发现比 v1多了五星评价功能，这不就是 ABTest吗，还是比较 6 的。

其他功能就不再演示了，主要有下面这些，官网都有：

- [Fault Injection](https://istio.io/docs/tasks/traffic-management/fault-injection.html)
- [Traffic Shifting](https://istio.io/docs/tasks/traffic-management/traffic-shifting.html)
- [Circuit Breaking](https://istio.io/docs/tasks/traffic-management/circuit-breaking.html)
- [Mirroring](https://istio.io/docs/tasks/traffic-management/mirroring.html)



之后打算用一下这个试一下，感觉还不错的样子，最后 友情提示一下官方推荐用 Google Kubernetes Engine，但是最好还是本地弄，反正我试了几次都是有问题的，可能是 RP 问题。



## 参考文档

1.[minikube 安装指南](https://kubernetes.io/docs/getting-started-guides/minikube/)

2.[istio 安装指南](https://istio.io/docs/setup/kubernetes/quick-start.html)







