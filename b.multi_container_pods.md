![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)
# 複数コンテナのPod (10%)

### 2つのコンテナを持つPodを作成し、どちらもイメージbusyboxとコマンド "echo hello; sleep 3600 "を実行します。2つ目のコンテナに接続し、「ls」を実行します

<details><summary>show</summary>
<p>

最も簡単な方法は、単一のコンテナでポッドを作成し、その定義を YAML ファイルに保存します


```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

コンテナ関連の値をコピー＆ペーストして、YAMLには以下の2つのコンテナが含まれるようにします

これらのコンテナは異なる名前にする必要があります

```YAML

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
# Connect to the busybox2 container within the pod
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit

# or you can do the above with just an one-liner
kubectl exec -it busybox -c busybox2 -- ls

# you can do some cleanup
kubectl delete po busybox
```

</p>
</details>

### 80番ポートに公開されたnginxコンテナを持つPodを作成します。busybox initコンテナを追加し、「wget -O /work-dir/index.html http://neverssl.com/online 」でページをダウンロードします。emptyDirタイプのボリュームを作成し、両方のコンテナでマウントします。nginxコンテナは「/usr/share/nginx/html」、initコンテナは「/work-dir」にマウントします。終わったら、作成したpodのIPを取得してbusybox podを作成し、"wget -O- IP" を実行します

<details><summary>show</summary>
<p>

最も簡単な方法は、単一のコンテナでポッドを作成し、その定義を YAML ファイルに保存します

```bash
kubectl run box --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

コンテナ関連の値をコピー＆ペーストして、YAMLにボリュームとinitContainerが含まれるようにします:

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
  - wget -O /work-dir/index.html http://neverssl.com/online
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
  initContainers: 
  - args: 
    - /bin/sh 
    - -c 
    - wget -O /work-dir/index.html http://neverssl.com/online 
    image: busybox 
    name: box 
    volumeMounts: 
    - name: vol 
      mountPath: /work-dir 
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts: 
    - name: vol 
      mountPath: /usr/share/nginx/html 
  volumes: 
  - name: vol 
    emptyDir: {} 
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Execute wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- $(kubectl get pod box -o jsonpath='{.status.podIP}')"

# you can do some cleanup
kubectl delete po box
```

</p>
</details>

