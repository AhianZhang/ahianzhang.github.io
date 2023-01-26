---
title: "Kubernetes 部署 Elasticsearch 和 Kibana"
date: 2023-01-25T15:51:36+08:00
tags: ["ElasticSearch"]
categories: ["搜索"]
---

Elastic 官方提供了 QuickStart 让我们简单快速的在本地部署 ElasticSearch，基于 [Kubernetes Operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) 模式，自动化的部署应用。

当前验证版本：

- ECK 2.6
-  Mac Docker Desktop 4.15.0
- Docker Engine: 20.10.21
- Kubernetes:  1.25.2

# 部署ECK

##  安装 CRD（custom resource definitions）

```shell
kubectl create -f https://download.elastic.co/downloads/eck/2.6.1/crds.yaml
```

## 安装 Operator 

```shell
kubectl apply -f https://download.elastic.co/downloads/eck/2.6.1/operator.yaml
```

## 监控 Operator 日志

```shell
kubectl -n elastic-system logs -f statefulset.apps/elastic-operator
```



# 部署 ElasticSearch

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 8.6.0
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
EOF
```

## 查看集群状态

```shell
kubectl get elasticsearch
```

## 查看 pod 

```shell
kubectl get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=quickstart'
```

## 外部访问

### 生成密码

```sh
PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
```

### 映射外部访问

```shell
kubectl port-forward service/quickstart-es-http 9200
```

### 验证

```sh
curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
```

 ```json
 {
   "name" : "quickstart-es-default-0",
   "cluster_name" : "quickstart",
   "cluster_uuid" : "PFBJFEyTSPyoF7-kMTYw0g",
   "version" : {
     "number" : "8.6.0",
     "build_flavor" : "default",
     "build_type" : "docker",
     "build_hash" : "f67ef2df40237445caa70e2fef79471cc608d70d",
     "build_date" : "2023-01-04T09:35:21.782467981Z",
     "build_snapshot" : false,
     "lucene_version" : "9.4.2",
     "minimum_wire_compatibility_version" : "7.17.0",
     "minimum_index_compatibility_version" : "7.0.0"
   },
   "tagline" : "You Know, for Search"
 }
 ```



# 部署 Kibana

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 8.6.0
  count: 1
  elasticsearchRef:
    name: quickstart
EOF
```

## 查看集群信息

```sh
kubectl get kibana
```

## 关联 ElasticSearch 实例

```sh
kubectl get pod --selector='kibana.k8s.elastic.co/name=quickstart'
```

## 访问 Kibana

### 查看服务状态

```sh
kubectl get service quickstart-kb-http
```

### 本地端口映射

```sh
kubectl port-forward service/quickstart-kb-http 5601
```

### 生成密码

```sh
kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
```

### 验证

访问 https://localhost:5601 elastic/密码 查看
