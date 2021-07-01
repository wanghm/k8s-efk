# k8s-efk

このワークショップはEFK (Elasticsearch, Fluentd, Kibana) スタックを Kubernetesクラスタへインストールし、アプリケーションログを取得する方法を説明します。

構成：

<image src=./images/EFK_architecture.png width=640>

## 0. helmをPCにインストール

手順： https://helm.sh/ja/docs/intro/install/

Mac:
```
brew install helm
```

Windows:
Helm コミュニティのメンバーが Chocolatey に Helm パッケージ のビルドを提供しました。 このパッケージは一般に最新です。

```
choco install kubernetes-helm
```

確認：

```
$ helm version
version.BuildInfo{Version:"v3.6.2", GitCommit:"ee407bdf364942bcb8e8c665f82e15aa28009b71", GitTreeState:"dirty", GoVersion:"go1.16.5"}

# helm repoを更新
helm repo update
helm repo list
```

## 1. Elasticsearchのインストール

```
$ kubectl create ns logging
$ helm repo add elastic https://helm.elastic.co
$ helm install elasticsearch elastic/elasticsearch --namespace logging --dry-run

$ helm install elasticsearch elastic/elasticsearch --namespace logging
NAME: elasticsearch
LAST DEPLOYED: Thu Jul  1 19:37:05 2021
NAMESPACE: logging
STATUS: deployed
REVISION: 1
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=logging -l app=elasticsearch-master -w
2. Test cluster health using Helm test.
  $ helm test elasticsearch

$ kubectl get pods --namespace=logging -l app=elasticsearch-master -w
NAME                     READY   STATUS     RESTARTS   AGE
elasticsearch-master-0   0/1     Init:0/1   0          2m10s
elasticsearch-master-1   0/1     Init:0/1   0          2m10s
elasticsearch-master-2   0/1     Pending    0          2m10s
elasticsearch-master-1   0/1     PodInitializing   0          2m39s
elasticsearch-master-1   0/1     Running           0          2m40s
elasticsearch-master-0   0/1     PodInitializing   0          3m8s
elasticsearch-master-0   0/1     Running           0          3m9s
elasticsearch-master-1   1/1     Running           0          4m32s
elasticsearch-master-0   1/1     Running           0          4m32s
```

## 2. fluentd-daemonsetのインストール

* Servie Account, cluster role, cluster rolebinnding 作成のコードを追加
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: logging-sa
  namespace: logging

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: logging-clusterrole
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: logging
  name: logging_clusterrolebindings
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: logging-clusterrole
subjects:
- kind: ServiceAccount
  name: logging-sa
  namespace: logging
```

```
kubectl apply -flogging_serviceaccount.yaml
kubectl apply -f fluentd-daemonset-elasticsearch.yaml
```

## 3. Kibanaのインストール

```
$ helm install kibana elastic/kibana -f kibana-values.yaml --namespace logging

NAME: kibana
LAST DEPLOYED: Thu Jul  1 20:16:32 2021
NAMESPACE: logging
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
