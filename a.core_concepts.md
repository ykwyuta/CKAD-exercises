![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)
# Core Concepts (13%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

### 'mynamespace'というnamespaceと、nginxのイメージのPodをmynamespaceに作る

<details><summary>show</summary>
<p>

```bash
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

</p>
</details>

### PodのYAMLファイルを作る

<details><summary>show</summary>
<p>

--dry-run=clientを指定すると、Kubernetesにリソースは作られません。-o yamlでYAMLを出力します。

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
  namespace: mynamespace
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

kubectl create -fでYAMLを元にリソースを作ります。

```bash
kubectl create -f pod.yaml
```

一行で上記の内容をやる場合は以下のようになります。

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n mynamespace -f -
```

</p>
</details>

### kubectlでbusyboxのPodを作り、そこでenvコマンドの実行結果を確認する

<details><summary>show</summary>
<p>

アウトプットを表示したらそのままPodを削除する場合

```bash
kubectl run busybox --image=busybox --command --restart=Never -it --rm -- env
```

コマンドを実行したあと、ログを確認する場合

```bash
kubectl run busybox --image=busybox --command --restart=Never -- env
kubectl logs busybox
```

</p>
</details>

### YAMLで busybox の podを作り、 そこでenvコマンドの実行結果を確認する

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
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
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

YAMLからPodを作成してログを確認します。

```bash
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>

### mynsというnamespaceを作らずにYAMLだけを作成する

<details><summary>show</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run=client
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: myns
spec: {}
status: {}
```

</p>
</details>

### myrqという1 CPU、1G memory、2 podsのResourceQuota のYAMLだけを作成する

<details><summary>show</summary>
<p>

hardの反対はsoftですが、ResourceQuotaにはsoftの制限は定義できません。制限を超えると即座に適用されます。

単純にcpuと定義した場合はrequests.cpuと同じ意味になります。

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: myrq
spec:
  hard:
    cpu: "1"
    memory: 1G
    pods: "2"
status: {}
```

</p>
</details>

### すべての namespaceのPodの一覧を取得する

<details><summary>show</summary>
<p>

```bash
kubectl get po --all-namespaces
```

以下のようにしても同じ結果になります。

```bash
kubectl get po -A
```
</p>
</details>

### nginxイメージのポート80を使うPod作成する

<details><summary>show</summary>
<p>

ちなみにこの--port=80はcontainerPort: 80に反映される項目ですが、この設定は利用するポートを明示するためのもので、指定があってもなくても挙動には影響しません。

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
```

</p>
</details>

### Change pod's image to nginx:1.7.1. Observe that the container will be restarted as soon as the image gets pulled

<details><summary>show</summary>
<p>

*Note*: The `RESTARTS` column should contain 0 initially (ideally - it could be any number)

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
kubectl get po nginx -w # watch it
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

*Note*: you can check pod's image by running

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Or just in one line:

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

</p>
</details>

### Get pod's YAML

<details><summary>show</summary>
<p>

```bash
kubectl get po nginx -o yaml
# or
kubectl get po nginx -oyaml
# or
kubectl get po nginx --output yaml
# or
kubectl get po nginx --output=yaml
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
# or
kubectl logs nginx --previous
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# or
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
```

</p>
</details>
