# k8s-efk

このワークショップはEFK (Elasticsearch, Fluentd, Kibana) スタックを Kubernetesクラスタへインストールし、アプリケーションログを取得する方法を説明します。

本Labの前提：　nginx ingress controller導入済み

nginx ingress controllerのEXTERNAL-IPの確認方法
```
$ kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   172.19.51.177    10.129.45.13   80:31546/TCP,443:31615/TCP   36h
ingress-nginx-controller-admission   ClusterIP      172.19.215.144   <none>         443/TCP                      36h
```

## 構成：

<image src=./images/EFK_architecture.png width=640>

## インストール
```
kubectl -f kube-logging.yaml
kubectl -f elasticsearch.yaml
kubectl -f kibana.yaml
kubectl -f fluentd.yaml
```
## 動作確認
```
# ingress作成
kubectl apply -f ingress-elasticsearch-kibana.yaml

# test log生成
kubectl apply -f test-counter.yaml
```
### ホスト名： kibana.localを作業PCのhostsファイルに追記
```
#Mac: 
sudo echo "<<ingress-nginx-controllerのEXTERNAL-IP>> kibana.local >> /etc/hosts"

# Windows:
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "<ingress-nginx-controllerのEXTERNAL-IP>`kibana.local" -Force
```
