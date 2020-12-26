![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/observability&empty)
# Observability (18%)

## Liveness and readiness probes

kubernetes.io > ドキュメント > タスク > Podとコンテナの設定 > [Liveness Probe、Readiness ProbeおよびStartup Probeを使用する](https://kubernetes.io/ja/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

### 'ls'コマンドを実行するだけのlivenessProbe付きnginxのPodを作成するYAMLをpod.yamlという名前で保存して、それを実行、probeの状態を確認したら削除してください

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
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
    livenessProbe: # Probeの定義です
      exec: # この行を追加します
        command: # commandの定義
        - ls # lsコマンド
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness # これを実行するとlivenessProbeが動作している事がわかります
kubectl delete -f pod.yaml
```

</p>
</details>

### pod.yamlを変更して、livenessProbeが5秒後に開始され、以降5秒毎に実行されるようにしてください。それを実行し、probeを確認し、削除してください

<details><summary>show</summary>
<p>

```bash
kubectl explain pod.spec.containers.livenessProbe # 正確な名前を取得します
```

```YAML
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
    livenessProbe: 
      initialDelaySeconds: 5 # この行を追加します
      periodSeconds: 5 # この行も追加します
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe po nginx | grep -i liveness
kubectl delete -f pod.yaml
```

</p>
</details>

### (80番ポートを公開して)nginxのpodを起動し、ポート80の'/'をHTTPでチェックするreadinessProbeを持たせて起動させてください、その後readinessProbeを確認して削除してください

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never --port=80 > pod.yaml
vi pod.yaml
```

```YAML
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
    ports:
      - containerPort: 80 # Note: readinessProbeはコンテナでそのライフサイクル中に渡って働きます。nginxが80番ポートを公開するので、containerPort: 80 はreadinessの稼働に直接必要ではありません。
    readinessProbe: # readinessProbeを定義します
      httpGet: # この行を追加します
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness # readnessの状態を確認します
kubectl delete -f pod.yaml
```

</p>
</details>

## Logging

### busyboxのPodを作成し、'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'を実行、そのログを取得してください

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # ログを参照します
```

</p>
</details>

## Debugging

### 'ls /notexist'を実行するbusyboxのPodを作成してください、そしてエラーが存在するか確認してください(もちろん、エラーになります)、エラーを確認した後Podを終了して消してください

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# それがエラーになっている事を確認します
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

</p>
</details>

### 'notexist'を実行するbusyboxのPodを作成し、それがエラーになる事を確認します(もちろんエラーです)、それを確認してPodを終了させたら、Podを猶予時間0で強制的に消してください

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- notexist
kubectl logs busybox # なにも表示されません！ コンテナは立ち上がりません
kubectl describe po busybox # eventsセクションにエラーが記録されています
# また...
kubectl get events | grep -i error # これでも同じようにエラーを確認できます
kubectl delete po busybox --force --grace-period=0
```

</p>
</details>

### NodeのCPUとメモリの利用率を取得してください ([metrics-server](https://github.com/kubernetes-incubator/metrics-server)が実行されている必要があります)

<details><summary>show</summary>
<p>

```bash
kubectl top nodes
```

</p>
</details>
