![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/services&empty)
# Services and Networking (13%)

### nginxという名前でnginxが稼働するPodを作成して、80番ポートを公開してください

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
# Podと同様に、serviceも作成されている事を確認します
```

</p>
</details>


### ClusterIPが作成された事を確認してください、endpointsも同様に確認してください

<details><summary>show</summary>
<p>

```bash
kubectl get svc nginx # services
kubectl get ep # endpoints
```

</p>
</details>

### Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget
### サービスのClusterIPを取得して、作業用busyboxPodからwgetでそのIPを'叩いて'ください

<details><summary>show</summary>
<p>

```bash
kubectl get svc nginx # IPを取得 (10.108.93.130のようなアドレスです)
kubectl run busybox --rm --image=busybox -it --restart=Never -- sh
wget -O- IP:80
exit
```

</p>
または
<p>

```bash
IP=$(kubectl get svc nginx --template={{.spec.clusterIP}}) # IPを取得 (10.108.93.130のようなアドレスです)
kubectl run busybox --rm --image=busybox -it --restart=Never --env="IP=$IP" -- wget -O- $IP:80 --timeout 2
# Tip: --timeout は任意です, しかし、コネクションが失敗した時により早く答えを得る助けになります (数分が数秒になります)
```

</p>
</details>

### Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.
### ClusterIPをNodePortに変更して同じサービスを作成して、NodePortのポートを取得してください。NodeのIPを利用してそのサービスを叩いたら、サービスを削除、Podを終了させてください

<details><summary>show</summary>
<p>

```bash
kubectl edit svc nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-06-25T07:55:16Z
  name: nginx
  namespace: default
  resourceVersion: "93442"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: 191e3dac-784d-11e8-86b1-00155d9f663c
spec:
  clusterIP: 10.97.242.220
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort # clusterIPからnodeportに変更します
status:
  loadBalancer: {}
```

```bash
kubectl get svc
```

```
# 出力:
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        1d
nginx        NodePort    10.107.253.138   <none>        80:31931/TCP   3m
```

```bash
wget -O- NODE_IP:31931 # もしDocker for Windows/Macに内蔵のKubernetesを利用しているなら、127.0.0.1で試してみてください
# もしminikubeを利用しているなら、minikube ipを実行して、192.168.99.117のようなnodeのIPを取得してください。
```

```bash
kubectl delete svc nginx # serviceを削除
kubectl delete pod nginx # podを削除
```
</p>
</details>

### fooという名前で'dgkanatsios/simpleapp'のイメージ(ホスト名を返す単純なサーバです)と3つのレプリカを持ったデプロイメントを作成してください。それに'app=foo'のラベルを付けて、8080番ポートからアクセスを受けるようにしてください。(サービスまだ作成しないでください)

<details><summary>show</summary>
<p>

```bash
kubectl create deploy foo --image=dgkanatsios/simpleapp --port=8080 --replicas=3
```
</p>
</details>

### PodのIPを取得してください。作業用のbusyboxPodを作成し、その8080番ポートを叩いてください

<details><summary>show</summary>
<p>


```bash
kubectl get pods -l app=foo -o wide # 'wide'オプションでIPを表示します
kubectl run busybox --image=busybox --restart=Never -it --rm -- sh
wget -O- POD_IP:8080 # Pod名で試行しないでください、失敗します
# 全てのIPを叩いて、ホスト名が異なる事を確認します
exit
```

</p>
</details>

### 作成したDeploymentをポート6262番で公開するServiceを作成して、その存在を確認し、エンドポイントをチェックしてください

<details><summary>show</summary>
<p>


```bash
kubectl expose deploy foo --port=6262 --target-port=8080
kubectl get service foo # ClusterIPと6262番ポートが表示されます
kubectl get endpoints foo # ポート8080でリッスンしている3つのレプリカのIPが表示されます
```

</p>
</details>

### 一時的なBusyboxPodを作成し、fooサービスへwgetでアクセスして、毎回異なったホスト名を返すことを確認してください。DeploymentとServiceを削除して、クラスタを掃除してください

<details><summary>show</summary>
<p>

```bash
kubectl get svc # fooサービスのClusterIPを取得します
kubectl run busybox --image=busybox -it --rm --restart=Never -- sh
wget -O- foo:6262 # DNSが使えます! 何度も試すと、異なったPodからレスポンスが帰ります
wget -O- SERVICE_CLUSTER_IP:6262 # ClusterIPでも動作します
# you can also kubectl logs on deployment pods to see the container logs
# DeploymentのPodにkubectl logsを利用すれば、コンテナのログを確認することもできます
kubectl delete svc foo
kubectl delete deploy foo
```

</p>
</details>

### nginxで2つのレプリカをもつDeploymentを作成し、それをClusterIP、80番ポートで公開してください。NetworkPolicyを作成し、'access: granted'のラベルが付いたpodからのみアクセスできるようにしてください

kubernetes.io > ドキュメント > コンセプト > Service、負荷分散とネットワーキング > [ネットワークポリシー](https://kubernetes.io/ja/docs/concepts/services-networking/network-policies/)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose ndeployment nginx --port=80

kubectl describe svc nginx # Podにあるセレクタ'app=nginx'を参照します
# または
kubectl get svc nginx -o yaml

vi policy.yaml
```

```YAML
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx # 名前を決めます
spec:
  podSelector:
    matchLabels:
      run: nginx # Podへのセレクタです
  ingress: # ingressのトラフィックを許可します
  - from:
    - podSelector: # Podから
        matchLabels: # このラベルが付与されているなら
          access: granted
```

```bash
# NetworkPolicyを作成
kubectl create -f policy.yaml

# NetworkPolicyが正しく作成されたかチェックします
# クラスタのネットワークプロバイダが、NetworkPolicyを正しくサポートするようにしてください (https://kubernetes.io/ja/docs/concepts/services-networking/network-policies/#%E5%89%8D%E6%8F%90%E6%9D%A1%E4%BB%B6)
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2                          # これは動作しません。 --timeoutオプションが指定されていますが、これはより早く結果を知る助けになります(数分から数秒になります)
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted -- wget -O- http://nginx:80 --timeout 2  # これは正しく動作します
```

</p>
</details>
