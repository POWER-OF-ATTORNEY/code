![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)
# マルチコンテナPod (10%)

### "echo hello; sleep 3600"を実行するbusyboxを2つ持つPodを作成してください。2つ目のコンテナにアクセスし'ls'を実行してください。

<details><summary>show</summary>
<p>

最も簡単な方法は、1つのコンテナを持つPodを作成するyamlファイルを作成することです。

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

コンテナに関する記述をコピー&ペーストして、最終的に2つのコンテナを持つようにします。(それぞれのコンテナは別の名前を持つ必要があります):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
# busybox2コンテナの中にアクセスします
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit

# もしくは、上記の操作をワンライナーで行います
kubectl exec -it busybox -c busybox2 -- ls

# 掃除をします
kubectl delete po busybox
```

</p>
</details>


### Create nginx pod exposed at port 80. Add an busybox init container which downloads the k8s page by "wget -O /work-dir/index.html http://kubernetes.io". Make a volume of type emptyDir and mount it in both pods. For nginx mount it on "/usr/share/nginx/html" and for the initcontainer use mount it on "/work-dir". When done, get the IP of the nginx pod and create a busybox pod and run wget -O- IP

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run web --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

Copy/paste the container related values, so your final YAML should contain the volume and the initContainer:

Volume:

```YAML
containers:
  - image: nginx
...
    volumeMounts:
    - name: vol
      mountPath: /usr/share/nginx/html
  volumes:
  - name: vol
    emptyDir: {}
```

initContainer:

```YAML
...
initContainers:
- args:
  - /bin/sh
  - -c
  - wget -O /work-dir/index.html http://kubernetes.io
  image: busybox
  name: box
  volumeMounts:
  - name: vol
    mountPath: /work-dir
```

In total you get:

```YAML

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: box
  name: box
spec:
  initContainers: #
  - args: #
    - /bin/sh #
    - -c #
    - wget -O /work-dir/index.html http://kubernetes.io #
    image: busybox #
    name: box #
    volumeMounts: #
    - name: vol #
      mountPath: /work-dir #
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts: #
    - name: vol #
      mountPath: /usr/share/nginx/html #
  volumes: #
  - name: vol #
    emptyDir: {} #
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP
kubectl get po -o wide

# Execute wget
kubectl run box --image=busybox --restart=Never -ti --rm -- /bin/sh -c "wget -O- IP"

# you can do some cleanup
kubectl delete po box
```

</p>
</details>

