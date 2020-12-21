![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/pod_design&empty)
# Podの設計 (20%)

## ラベルとアノテーション
kubernetes.io > ドキュメント > コンセプト > 概要 > [ラベル(Labels)とセレクター(Selectors)](https://kubernetes.io/ja/docs/concepts/overview/working-with-objects/labels/#label-selectors)

### nginx1,nginx2,nginx3という名前でPodを3つ作成しましょう。全てのPodに'app=v1'のラベルを付けてください。

<details><summary>show</summary>
<p>

```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```

</p>
</details>

### Podのラベルを全て出力してください

<details><summary>show</summary>
<p>

```bash
kubectl get po --show-labels
```

</p>
</details>

### Pod'nginx2'のラベルを'app=v2'に変更してください

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx2 app=v2 --overwrite
```

</p>
</details>

### Podの'app'ラベルを取得してください (ラベルの行を表示してください)

<details><summary>show</summary>
<p>

```bash
kubectl get po -L app
# または
kubectl get po --label-columns=app
```

</p>
</details>

### 'app=v2'のラベルを持つPodのみ出力してください。

<details><summary>show</summary>
<p>

```bash
kubectl get po -l app=v2
# または
kubectl get po -l 'app in (v2)'
# もしくは
kubectl get po --selector=app=v2
```

</p>
</details>

### 先ほど作成したPodの'app'ラベルを削除してください。

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx1 nginx2 nginx3 app-
# もしくは
kubectl label po nginx{1..3} app-
# または
kubectl label po -l app app-
```

</p>
</details>

### 'accelerator=nvidia-tesla-p100'のラベルを持つNodeにデプロイされるPodを作成してください。
Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

<details><summary>show</summary>
<p>

Nodeにラベルを追加する:

```bash
kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels
```

PodのYamlで 'nodeSelector' プロパティを利用できます:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
  nodeSelector: # add this
    accelerator: nvidia-tesla-p100 # the selection label
```

YAMLの何処に設定するかは、下記のコマンドで簡単に見つかりますout where in the YAML it should be placed by:

```bash
kubectl explain po.spec
```

または:
node affinity を使う(https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/#schedule-a-pod-using-required-node-affinity)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100
  containers:
    ...
```

</p>
</details>

### Pod nginx1、nginx2、nginx3 に "description='my description'" というアノテーションを付ける

<details><summary>show</summary>
<p>


```bash
kubectl annotate po nginx1 nginx2 nginx3 description='my description'

または

kubectl annotate po nginx{1..3} description='my description'
```

</p>
</details>

### Pod nginx1 のアノテーションを表示する

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx1 | grep -i 'annotations'

# または

kubectl get pods -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

`| grep` を使用する代わりに、jsonPathを使う事もできます `kubectl get po nginx1 -o jsonpath='{.metadata.annotations}{"\n"}'`

</p>
</details>

### 3つのPodから、アノテーションを取り除いてください

<details><summary>show</summary>
<p>

```bash
kubectl annotate po nginx{1..3} description-
```

</p>
</details>

### これらのPodを削除して、クラスタを掃除します

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx{1..3}
```

</p>
</details>

## Deployments

kubernetes.io > Documentation > Concepts > Workloads > Controllers > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### イメージがnignx:1.7.8で、レプリカ2つ持ち、80番ポートを公開したnginxというデプロイメントを作成してください。(serviceは作らないでください)
### Create a deployment with image nginx:1.7.8, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run=client -o yaml > deploy.yaml
vi deploy.yaml
# フィールドを 1 から 2 に変更します
# deploy.yamlのコンテナspecにこの記述を追加して保存します
# add this section to the container spec and save the deploy.yaml file
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

もしくは、以下のようにします

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run=client -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.7.8/image: nginx:1.7.8\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

or, 
```bash
kubectl create deploy nginx --image=nginx:1.7.8 --replicas=2 --port=80
```

</p>
</details>

### このdeploymentのYAMLを表示します

<details><summary>show</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### View the YAML of the replica set that was created by this deploymentで作成されたReplicaSetのYAMLを表示します

<details><summary>show</summary>
<p>

```bash
kubectl describe deploy nginx # Eventsセクションの'NewReplicaSet'プロパティでReplicaSetの名前を確認できます
# または、直接ReplicaSetを見つけます
kubectl get rs -l run=nginx # 'run' コマンドでdeploymentを作成した場合
kubectl get rs -l app=nginx # 'create' コマンドでdeploymentを作成した場合
# kubectl get rs で取得する事もできます
kubectl get rs nginx-7bf7478b77 -o yaml
```

</p>
</details>

### 1つのPodのYAMLを表示してください

<details><summary>show</summary>
<p>

```bash
kubectl get po # 全てのPodを取得します
# もしくは、直接Podを取得できます:
kubectl get po -l run=nginx # 'run' コマンドでdeploymentを作成した場合
kubectl get po -l app=nginx # 'create' コマンドでdeploymentを作成した場合
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

</p>
</details>

### deployment の rollout の進捗を確認する

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### nginx のイメージを nginx:1.7.9 に変更してください

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.7.9
# もしくは...
kubectl edit deploy nginx # .spec.template.spec.containers[0].imageを変更します
```

'kubectl set image' コマンドのシンタックスは `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

</p>
</details>

### Rollout の履歴を確認し、Replicaが正常かどうか確認してください

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # 新しいレプリカが作成されているか確認
kubectl get po
```

</p>
</details>

### 最後のrolloutを戻して、新しいPodが古いイメージで動作しているかを確認してください

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# 少し待ちます
kubectl get po # select one 'Running' Pod
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # nginx:1.7.8 である必要があります
```

</p>
</details>

### nginx:1.91という間違ったイメージで意図的にdeploymentを更新します

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# または
kubectl edit deploy nginx
# image を nginx:1.91 に変更します
# vim tip: '/image' と(クォート無しで)入力してEnterを押すと、素早く移動できます
```

</p>
</details>

### Rolloutが何処かおかしい事を確認します

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
# または
kubectl get po # 'ErrImagePull'が見えるハズです
```

</p>
</details>


### Deployment を2番目のリビジョンに戻して、nginx:1.7.9イメージで動作している事を確認してください

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # 全て問題無い必要があります
```

</p>
</details>

### 4番目のリビジョンの詳細を確認してください


<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # You'll also see the wrong image displayed here
```

</p>
</details>

### Deploymentをレプリカ数5にスケールさせてください

<details><summary>show</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
```

</p>
</details>

### DeploymentをPod数が5から10個になるようオートスケールさせてください。しきい値はCPU使用率が80%以上です

<details><summary>show</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
```

</p>
</details>

### Deploymentのrolloutを一時停止させてください

<details><summary>show</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### イメージをnginx:1.9.1にアップデートしてそれがrolloutを停止させた以降デプロイされてない事を確認してください

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.9.1
# または
kubectl edit deploy nginx
# imageをnginx:1.9.1に変更
kubectl rollout history deploy nginx # no new revision
```

</p>
</details>

### Rolloutのデプロイを再開し、イメージnginx:1.9.1がデプロイされる事を確認してください

<details><summary>show</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # 最新のrevision番号を挿入してください
```

</p>
</details>

### Deployment と horizontal pod autoscaler を削除してください
<details><summary>show</summary>
<p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

#Or
kubectl delete deploy/nginx hpa/nginx
```
</p>
</details>

## Jobs

### "perl -Mbignum=bpi -wle 'print bpi(2000)'"のコマンドと引数を実行するjobをperlイメージで作成してください

<details><summary>show</summary>
<p>

```bash
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

</p>
</details>

### jobが終了するまで待って、出力を取得してください。

<details><summary>show</summary>
<p>

```bash
kubectl get jobs -w # 'SUCCESSFUL' が 1 になるまで待ちます。(will take some time, perl image might be big)
kubectl get po # get the pod の名前を取得します
kubectl logs pi-**** # 円周率を出力します
kubectl delete job pi
```

</p>
</details>

### busyboxイメージで、'echo hello;sleep 30;echo world'を実行するjobを作成してください

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### Podのログを取得してください (30秒待つ必要があります)

<details><summary>show</summary>
<p>

```bash
kubectl get po # find the job pod
kubectl logs busybox-ptx58 -f # follow the logs
```

</p>
</details>

### jobの状態と詳細、ログを表示してください。

<details><summary>show</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### jobを削除してください

<details><summary>show</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### jobを作成 しかし もし30秒以上終了しない場合は、kubernetesによって自動的に終了する事を確認する。
### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>
  
```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```
  
job.spec.activeDeadlineSeconds=30 を追記

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```
</p>
</details>

### 同じjobを5つ作成してください。少し待って、ステータスと削除されているかを確認してください
### Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Add job.spec.completions=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # この行を追加
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

正常に完了した事を確認する:

```bash
kubectl get job busybox -w # 2分半かかります
kubectl delete jobs busybox
```

</p>
</details>

### 同じjobを5つ並列に実行してください。
### Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```bash
vi job.yaml
```

job.spec.parallelism=5 を追加

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # この行を追加
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

並列実行したjobが終了するまで少しかかります(30秒以上)

```bash
kubectl delete job busybox
```

</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [自動化されたタスクをCronJobで実行する (en)](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### busyboxイメージを使って、'date; echo Hello from the Kubernetes cluster'と標準出力に書き込むジョブを"*/1 * * * *"にスケジュールしてください


<details><summary>show</summary>
<p>

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### そのログを見て、ジョブを削除してください

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # podには親のジョブを示すラベルが付いている事に注意してください
kubectl logs busybox-1529745840-m867r
# Kubernetesは新しいcronjobごとに新しいjob/podを生成する事に注意してください
kubectl delete cj busybox
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its schedule.
### busyboxイメージを使って、'date; echo Hello from the Kubernetes cluster'と毎分標準出力に書き込むジョブを作成してください。ただしスケジュールされてから17秒以上かかる場合は終了する必要があります

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
cronjob.spec.startingDeadlineSeconds=17 を追記

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 # この行を追記
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>
