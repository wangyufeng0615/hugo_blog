---
title: "在k8s中部署EFK日志收集方案"
date: 2019-06-06T00:00:00+08:00
draft: false
---

最近研究了一个事，就是怎么在kubernetes集群中部署一套日志采集系统。

在研究的过程中，我发现了一篇DigitalOcean的教程，这篇教程讲的非常详细，手把手教学，非常推荐。如果想在k8s集群中搭建EFK(ElasticSearch、Fluentd、Kibana)日志采集方案，看这一篇文章就足够了。

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes#step-2-%E2%80%94-creating-the-elasticsearch-statefulset

但在部署的过程中，也遇到了一些小坑，分享出来供大家参考。

默认的ES内存配置，在数据量较大的情况下，可能不够用，建议改大，例如"-Xms2048m -Xmx2048m"
如果你想把es和kibana部署在master节点上，默认情况下是不行的，因为master节点不允许被调度，需要修改污点和容忍的策略。例如，可以修改master node的调度策略为”尽量不调度”。