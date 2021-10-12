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

<image src=./images/EFK.png width=640>

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
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-kibana
  namespace: kube-logging
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: "kibana.local" 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kibana
            port:
              number: 5601
  - host: "elastic.local" 
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: elasticsearch
            port:
              number: 9200
```

### test log生成用Podを作成
```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c, 'i=0; while true; do echo "This is demo log $i: $(date)"; i=$((i+1)); sleep 1; done']
```
```
kubectl apply -f test-counter.yaml
```

### ホスト名： kibana.localを作業PCのhostsファイルに追記
```
#Mac: 
sudo echo "<<ingress-nginx-controllerのEXTERNAL-IP>> kibana.local >> /etc/hosts"

# Windows:
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "<ingress-nginx-controllerのEXTERNAL-IP>`kibana.local" -Force
```
### ブラウザでkibana.localにアクセス

DiscoveryでIndex Pattern設定
Filterでkubernetes.pod_nameを選択
<img src=./images/kibana_1.png width=768>
