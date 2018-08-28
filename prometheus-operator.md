# Prometheus Operator 快速安装部署流程
> 作者:流火夏梦


> 时间:2018/8/28

##### 针对国内k8s集群安装部署监控流程中k8s全局监控策略方案的提供，鉴于Operator 部署简单便捷，作为实战部署经验，这里做一个工作报告的形式，阐述Prometheus Operator架构，以及快速部署的流程。

## Prometheus Operator  架构

![](https://i.imgur.com/aA6xMu4.png)


### Prometheus的特点： ###
- 1、多维数据模型（时序列数据由metric名和一组key/value组成）
- 2、在多维度上灵活的查询语言(PromQl)
- 3、不依赖分布式存储，单主节点工作.
- 4、通过基于HTTP的pull方式采集时序数据
- 5、可以通过中间网关进行时序列数据推送(pushing)
- 6、目标服务器可以通过发现服务或者静态配置实现
- 7、多种可视化和仪表盘支持

**prometheus 相关组件，Prometheus生态系统由多个组件组成，其中许多是可选的：**


- 1、Prometheus 主服务,用来抓取和存储时序数据
- 2、client library 用来构造应用或 exporter 代码 (go,java,python,ruby)
- 3、push 网关可用来支持短连接任务
- 4、可视化的dashboard (两种选择,promdash 和 grafana.目前主流选择是 grafana.)
  一些特殊需求的数据出口(用于HAProxy, StatsD, Graphite等服务)
- 5、实验性的报警管理端(alartmanager,单独进行报警汇总,分发,屏蔽等 )

**promethues 的各个组件基本都是用 golang 编写,对编译和部署十分友好.并且没有特殊依赖.基本都是独立工作。**



## github项目目录结构

![](https://i.imgur.com/wkhrYCp.png)

![](https://i.imgur.com/FVoU2Et.png)

### 根据从github下载下来的项目目录结构我们开始进行项目部署。


## 开始部署
#### 首先进行operator的管理安装


>kubectl apply -f bundle.yaml

![](https://i.imgur.com/kxJ4MPV.png)


#### 部署其他监控组件以及监控界面

>kubectl apply -f kube-prometheus/manifests/

![](https://i.imgur.com/VEs5Ywi.png)

看到这些pod 正常runing就证明了初步胜利的曙光。

查看服务是否正常开启:

![](https://i.imgur.com/Q8jMhsQ.png)


根据开放的服务端口进行服务访问，验证服务是否正常。

>![](https://i.imgur.com/xGjiDBE.png)

>![](https://i.imgur.com/umYfB91.png)

>![](https://i.imgur.com/fKVWusw.png)

特别提醒:我们注意报警栏目中的报警信息项，对应于prometheus-operator-/prometheus/kube-prometheus/manifests/prometheus-rules.yaml   这个文件中的规则。


**举例说明:下文就是报警信息的一种举例说明**


    alert: PrometheusTargetScrapesDuplicate
      annotations:
        description: '{{$labels.namespace}}/{{$labels.pod}} has many samples rejected
          due to duplicate timestamps but different values'
        summary: Prometheus has many samples rejected
      expr: |
        increase(prometheus_target_scrapes_sample_duplicate_timestamp_total{job="prometheus-k8s"}[5m]) > 0
      for: 10m
      labels:
        severity: warning

>![](https://i.imgur.com/MvccqNb.png)



之后我们查看grafana界面信息:
>![](https://i.imgur.com/ClhrF4Q.png)

>![](https://i.imgur.com/4WDZwgY.png)


#### 部署thanos在集群中的部署文档，进行如下

>kubectl apply -f example/thanos

之后就会发现及集群中将会新增加如下pod 和service
>![](https://i.imgur.com/w05I88J.png)

>![](https://i.imgur.com/3t9Fa3U.png)

访问应用服务端口


>![](https://i.imgur.com/z5IgGqN.png)


## 总结：
下一步数据持久化，进行数据持久存储操作。通过连接thanos将普罗米修斯的数据持久化的进行存储，以备后续查询操作。