![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)
# コアコンセプト (13%)

kubernetes.io > ドキュメント > リファレンス > kubectl CLI > [kubectlチートシート](https://kubernetes.io/ja/docs/reference/kubectl/cheatsheet/)

kubernetes.io > ドキュメント > タスク > 監視、ログ、デバック > [実行中のコンテナへのシェルを取得する](https://kubernetes.io/ja/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > ドキュメント > タスク > クラスター内アプリケーションへのアクセス > [複数のクラスターへのアクセスを設定する](https://kubernetes.io/ja/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > ドキュメント > タスク > クラスター内アプリケーションへのアクセス > [クラスターへのアクセス(EN)](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) APIを利用した

kubernetes.io > ドキュメント > タスク > クラスター内アプリケーションへのアクセス > [ポートフォワーディングを利用したクラスター内アプリケーションへのアクセス(EN)](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

<details><summary>原文(英語)</summary>

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

</details>

### 'mynamespace' という名前でNamespaceを作成し、'nginx'という名前でnginxが稼動するPodを作成してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

</p>
</details>

### さっき作成したPodをYAMLファイルを利用して作成してください。Create the pod that was just described using YAML

<details><summary>回答例</summary>
<p>

YAMLファイルの簡単な作成方法:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -n mynamespace -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml -n mynamespace
```

変わりに、ワンライナーでも実行できます。

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n mynamespace -f -
```

</p>
</details>

### busyboxが稼動するPodを作成(kubectlコマンドを利用してください)して、"env"コマンドを実行、出力を確認してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl run busybox --image=busybox --command --restart=never -it -- env # -it オプションを利用すると出力を直接取得できます。
# もしくは、-itオプション無しで実行して
kubectl run busybox --image=busybox --command --restart=never -- env
# ログを確認しましょう。
kubectl logs busybox
```

</p>
</details>

### busyboxのPodを作成(yamlを利用)して、"env"コマンドを実行、出力を確認してください。

<details><summary>回答例</summary>
<p>

```bash
# yamlテンプレートを以下のコマンドで作成します。
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml
# 中身を確認しましょう。
cat envpod.yaml
```

```yaml
apiversion: v1
kind: pod
metadata:
  creationtimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnspolicy: clusterfirst
  restartpolicy: never
status: {}
```

```bash
# これをapplyして、ログを確認しましょう。
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>

### 新しいnamespace 'myns'を作成するyamlを、ネームスペースを作成する事なく出力してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run=client
```

</p>
</details>

### 'myrq'という名前で、1CPU、メモリ1G、ポッド数2のハードリミットを持つresourcequotaを作成するYAMLファイルをPodを作成する事なく出力してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

</p>
</details>

### 全namespacesのPod一覧を取得してください

<details><summary>回答例</summary>
<p>

```bash
kubectl get po --all-namespaces
```
Alternatively 

```bash
kubectl get po -A
```
</p>
</details>

### nginxが稼動するPodを作成し、80番ポートへの通信を許可してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=never --port=80
```

</p>
</details>

### Podのイメージをnginx:1.7.1にしてください。Imageが取得された後すぐにPodが削除されて再生成されるのを観測してください。

<details><summary>回答例</summary>
<p>

*Note*: The `RESTARTS` column should contain 0 initially (ideally - it could be any number)

```bash
# kubectl set image pod/pod_name container_name=image_name:tag
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe po nginx # 'container will be killed and recreated' というログを確認できます
kubectl get po nginx -w # 監視
```

*Note*: some time after changing the image, you should see that the value in the `RESTARTS` column has been increased by 1, because the container has been restarted, as stated in the events shown at the bottom of the `kubectl describe pod` command:

```
Events:
  Type    Reason     Age                  From               Message
  ----    ------     ----                 ----               -------
[...]
  Normal  Killing    100s                 kubelet, node3     Container pod1 definition changed, will be restarted
  Normal  Pulling    100s                 kubelet, node3     Pulling image "nginx:1.7.1"
  Normal  Pulled     41s                  kubelet, node3     Successfully pulled image "nginx:1.7.1"
  Normal  Created    36s (x2 over 9m43s)  kubelet, node3     Created container pod1
  Normal  Started    36s (x2 over 9m43s)  kubelet, node3     Started container pod1
```

*note*: 動作しているイメージのチェック

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### 先程作成したNginxPodのIPアドレスを取得して、一時的なbusyboxのイメージからnginxの'/'をwgetで取得してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl get po -o wide # IPを取得、'10.1.1.131'のようなアドレスが取得できます。
# 一時的なbusyboxPodを作成
kubectl run busybox --image=busybox --rm -it --restart=never -- wget -o- 10.1.1.131:80
```

もしくは、このようにより多くのオプションを利用します:
alternatively you can also try a more advanced option:

```bash
# nginxPodのIPを取得
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# 一時的なbusyboxPodを作成
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Or just in one line:

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

</p>
</details>

### 稼動中のPodのYAMLを取得してください

<details><summary>回答例</summary>
<p>

```bash
kubectl get po nginx -o yaml
# または
kubectl get po nginx -oyaml
# もしくは
kubectl get po nginx --output yaml
# あるいは
kubectl get po nginx --output=yaml
```

</p>
</details>

### 障害の可能性がわかるようなPodの情報を取得してください。(例:Podが起動していない)

<details><summary>回答例</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Podのログを取得してください

<details><summary>回答例</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### もし、Podがクラッシュして再起動している場合は、一つ前のインスタンスについてのログを取得してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl logs nginx -p
```

</p>
</details>

### Execute a simple shell on the nginx pod
nginxのPod上で、シェルを実行してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

</p>
</details>

### 'hello world'と出力した後終了するbusyboxのPodを作成してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# または
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### 同じ事をします、今度は処理の終了後にPodが自動的に削除されるようにしてください。

<details><summary>回答例</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # 何も表示されなければOK (^ ^)
```

</p>
</details>

### nginxのPodを作成して、環境変数'var1=val1'を設定してください。その後、Podの中で環境変数の存在を確認してください。

<details><summary>回答例</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# その後
kubectl exec -it nginx -- env
# または
kubectl exec -it nginx -- sh -c 'echo $var1'
# もしくは
kubectl describe po nginx | grep val1
# あるいは
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
```

</p>
</details>
