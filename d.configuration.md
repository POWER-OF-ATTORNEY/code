![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/configuration&empty)
# Configuration (18%)

## ConfigMaps

kubernetes.io > ドキュメント > タスク > Podとコンテナの設定 > [Podを構成してConfigMapを使用する](https://kubernetes.io/ja/docs/tasks/configure-pod-container/configure-pod-configmap/)

### configという名前で、foo=lala,foo2=loloの値を持つconfigmapを作成してください

<details><summary>show</summary>
<p>

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

</p>
</details>

### 設定した値を表示してください

<details><summary>show</summary>
<p>

```bash
kubectl get cm config -o yaml
# または
kubectl describe cm config
```

</p>
</details>

### configmapをファイルから作成して表示させてください

下記コマンドでファイルを作成します

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml
```

</p>
</details>

### Create and display a configmap from a .env file

下記コマンドでファイルを作成します

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml
```

</p>
</details>

### Create and display a configmap from a file, giving the key 'special'
### 'special'というキーで、ファイルからconfigmapを作成してください

下記コマンドでファイルを作成します

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4
kubectl get cm configmap4 -o yaml
```

</p>
</details>

### 'options'という名前で、var5=val5という値をもつconfigMapを作成して、nginxが稼働するpodを作成し、'option'環境変数から'val5'の値を参照してください

<details><summary>show</summary>
<p>

```bash
kubectl create cm options --from-literal=var5=val5
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
    env:
    - name: option # 環境変数の名前
      valueFrom:
        configMapKeyRef:
          name: options # configMapの名前
          key: var5 # configMap内の値の名前
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep option # 'option=val5'が表示されます
```

</p>
</details>

### 'var6=val6','var7=val7'をもつconfigMap'anotherone'を作成し、新しいnginxのPodから環境変数として読み込んでください

<details><summary>show</summary>
<p>

```bash
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run --restart=Never nginx --image=nginx -o yaml --dry-run=client > pod.yaml
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
    envFrom: # 前問は'env'でしたが、違う値です
    - configMapRef: # 前問は'configMapKeyRef'でしたが、違う値です
        name: anotherone # configMapの名前
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env 
```

</p>
</details>

### 'var8=val8','var9=val9'の値をもつ'cmvolume'というconfigMapを作成し、nginxが稼働するpodの/etc/lala内にボリュームとして読み込んでください。そして、/etc/lalaディレクトリ内を'ls'で表示してください

<details><summary>show</summary>
<p>

```bash
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
  volumes: # 追加するvolumeのリスト
  - name: myvolume # ただの名前です、Pod内で参照します
    configMap:
      name: cmvolume # configMapの名前です
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # volumeマウントをここに列挙します
    - name: myvolume # pod.spec.volumes.nameに定義した名前です
      mountPath: /etc/lala # コンテナ内の何処にマウントするかの定義です
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # var8 var9 と表示されます
cat var8 # val8 と表示されます
```

</p>
</details>

## SecurityContext

kubernetes.io > ドキュメント > タスク > Podとコンテナの設定 > [Configure a Security Context for a Pod or Container (en)](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### UID 101で実行されるnginxのPodのYAMLを作成してください。実際にPodを作る必要はありません

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
  securityContext: # この行を追加します
    runAsUser: 101 # UIDを記述します
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

### "NET_ADMIN"と"SYS_TIME"の機能が付加さたnginxのPodのYAMLを作成してください

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
    securityContext: # この行を追加
      capabilities: # これを追加
        add: ["NET_ADMIN", "SYS_TIME"] # これも追加
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

## Requests and limits

kubernetes.io > ドキュメント > タスク > Podとコンテナの設定 > [コンテナおよびPodへのCPUリソースの割り当て](https://kubernetes.io/ja/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi
### cpu=100m,memory=256Miを要求して、cpu=200m,memory=512Miのリソース制限をもつnginxのPodを作成してください

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```

</p>
</details>

## Secrets

kubernetes.io > ドキュメント > コンセプト > 設定 > [Secrets](https://kubernetes.io/ja/docs/concepts/configuration/secret/)

kubernetes.io > ドキュメント > タスク > アプリケーションへのデータ注入 > [Distribute Credentials Securely Using Secrets(en)](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### mysecretという名前で、password=mypassの値をもつSecretを作成してください

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>

### Create a secret called mysecret2 that gets key/value from a file
### ファイルからキー/値を取得して、mysecret2という名前のSecretを作成してください

adminという内容で、usernameというファイルを作成します:

```bash
echo -n admin > username
```

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
```

</p>
</details>

### mysecret2の値を取得してください

<details><summary>show</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml
echo YWRtaW4K | base64 -d # MACでは -D にしてください、値がデコードされて'admin'と表示されます
```

もしくは:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}{"\n"}' | base64 -d  # MACでは -D にしてください
```

</p>
</details>

### mysecret2を/etc/foo以下にボリュームとしてマウントしてあるnginxのpodを作成してください

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
  volumes: # volumesを定義
  - name: foo # この名前は後で参照します
    secret: # secretが必要です
      secretName: mysecret2 # secretの名前 - Podの作成時に存在している必要があります
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # volumeのマウントを定義します
    - name: foo # pod.spec.volumesの名前
      mountPath: /etc/foo # マウントパス
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx /bin/bash
ls /etc/foo  # usernameと表示されます
cat /etc/foo/username # adminと表示されます
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'
### 作成したPodを削除して、新しく'USERNAME'という環境変数にmysecret2のusernameが設定されたnginxのPodを作成してください

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
    env: # 環境変数の定義です
    - name: USERNAME # 名前です
      valueFrom:
        secretKeyRef: # secretを参照
          name: mysecret2 # secretの名前
          key: username # secret内のキーの値
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # 'admin'と表示されます
```

</p>
</details>

## ServiceAccounts

kubernetes.io > ドキュメント > タスク > Podとコンテナの設定 > [Configure Service Accounts for Pods(en)](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### すべてのネームスペース内にある、すべてのサービスアカウントを表示してください

<details><summary>show</summary>
<p>

```bash
kubectl get sa --all-namespaces
```
もしくは;

```bash
kubectl get sa -A
```

</p>
</details>

### 'myuser'という名前のサービスアカウントを新しく作成してください

<details><summary>show</summary>
<p>

```bash
kubectl create sa myuser
```

または:

```bash
# テンプレートを簡単に入手します
kubectl get sa default -o yaml > sa.yaml
vim sa.yaml
```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```

```bash
kubectl create -f sa.yaml
```

</p>
</details>

### 'myuser'サービスアカウントを利用して、nginxのpodを作成してください

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --serviceaccount=myuser -o yaml --dry-run=client > pod.yaml
kubectl apply -f pod.yaml
```

または手動で追加します:

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
  serviceAccountName: myuser # pod.spec.serviceAccountNameを利用して追加します
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
kubectl create -f pod.yaml
kubectl describe pod nginx # myuser-token-***** という新しいシークレットがマウントされている事が確認できます
```


</p>
</details>
