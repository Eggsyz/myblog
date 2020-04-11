---
title: "区块链Fabric监控部署"
date:  2019-09-28 10:33:03
categories: 区块链
tags:
- Prometheus
---

本文主要介绍区块链Fabric监控工具——Prometheus的使用。

## Prometheus

### Prometheus概述

#### 什么是Prometheus?

Prometheus是由SoundCloud开发的开源监控报警系统和时序列数据库(TSDB)。Prometheus使用Go语言开发，是Google BorgMon监控系统的开源版本。
####  Prometheus的特点

+ 多维度数据模型。
+ 灵活的查询语言。
+ 不依赖分布式存储，单个服务器节点是自主的。
+ 通过基于HTTP的pull方式采集时序数据。
+ 可以通过中间网关进行时序列数据推送。
+ 通过服务发现或者静态配置来发现目标服务对象。
+ 支持多种多样的图表和界面展示，比如Grafana等。

### fabric1.4 版本监控

Fabric1.4版本是一个里程碑式的版本，是首个LTS的版本（Long Term Support的版本），fabric提供了基于Prometheus这种拉式模型和基于StatsD这种推式模型的两种方式来获取指标数据。接下来本文将介绍如何使用Prometheus监控fabric。

#### fabric指标收集及配置

orderer.yml/peer.yml
```yaml
metrics:
    # metrics provider is one of statsd, prometheus, or disabled
    provider: disabled

    # statsd configuration
    statsd:
        # network type: tcp or udp
        network: udp

        # statsd server address
        address: 127.0.0.1:8125

        # the interval at which locally cached counters and gauges are pushed
        # to statsd; timings are pushed immediately
        writeInterval: 10s

        # prefix is prepended to all emitted statsd metrics
        prefix:
```
其中，配置prometheus只需要配置provider为prometheus,并且，由于采用拉的模式，需要peer和orderer提供对外的端口。具体配置为：
orderer.yml/peer.yml
```yaml
Operations:
    # host and port for the operations server
    ListenAddress: 127.0.0.1:8443

    # TLS configuration for the operations endpoint
    TLS:
        # TLS enabled
        Enabled: false

        # Certificate is the location of the PEM encoded TLS certificate
        Certificate:

        # PrivateKey points to the location of the PEM-encoded key
        PrivateKey:

        # Most operations service endpoints require client authentication when TLS
        # is enabled. ClientAuthRequired requires client certificate authentication
        # at the TLS layer to access all resources.
        ClientAuthRequired: false

        # Paths to PEM encoded ca certificates to trust for client authentication
        ClientRootCAs: []

```
上述配置默认不提供监控。
因此，只需要在启动fabric网络时修改peer-base.yaml文件，即可将指标对外提供。可通过ip:port/metrics查看具体的值。
+ peer-base添加下列环境变量及映射端口
```yaml
environment:
  - CORE_METRICS_PROVIDER=prometheus
  - CORE_OPERATIONS_LISTENADDRESS=0.0.0.0:8443
ports:
  - 8443:8443
```
+ orderer-base添加下列环境变量及映射端口
```yaml
environment:
  - ORDERER_METRICS_PROVIDER=prometheus
  - ORDERER_OPERATIONS_LISTENADDRESS=0.0.0.0:8443
ports:
  - 8444:8443
```
#### prometheus搭建部署及配置

通过docker部署prometheus。
+ 修改prometheus配置
```yaml
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'orderer.example.com'
    static_configs:
    - targets: ['192.168.101.198:8443']
  - job_name: 'peer0.org1.example.com'
    static_configs:
    - targets: ['192.168.101.198:8444']
  - job_name: 'peer0.org2.example.com'
    static_configs:
    - targets: ['192.168.101.198:8446']
```
+ 启动prometheus
```shell
docker run --name prometheus -d -v prometheus.yml:/etc/prometheus/prometheus.yml -p 9090:9090 prom/prometheus
```
+ 查看
```shell
http://192.168.9.96:9090
```