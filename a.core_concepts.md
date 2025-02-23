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

### nginx Podのイメージを nginx:1.7.1. に変更し、Podが変更される過程を確認する

<details><summary>show</summary>
<p>


`kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG`のように指定することでimageを変更することができます。

```bash
kubectl set image pod/nginx nginx=nginx:1.7.1
```

コンテナが削除されて再作成される過程を確認します。

```bash
kubectl describe po nginx
kubectl get po nginx -w
```

以下ような結果になります。

```
Events:
  Type    Reason   Age                  From     Message
  ----    ------   ----                 ----     -------
  Normal  Killing  9m29s                kubelet  Container nginx definition changed, will be restarted
  Normal  Pulling  9m29s                kubelet  Pulling image "nginx:1.7.1"
  Normal  Pulled   9m19s                kubelet  Successfully pulled image "nginx:1.7.1" in 10.499051067s
  Normal  Created  9m19s (x2 over 10h)  kubelet  Created container nginx
  Normal  Started  9m19s (x2 over 10h)  kubelet  Started container nginx
```

実行しているPodのimageが何になっているかは以下で確認できます。

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}'
```

</p>
</details>

### Nginx Podを作成してそのIPを確認し、一時的に作成した busybox イメージ から wget でアクセスする

<details><summary>show</summary>
<p>

Nginx Podを作成してそのIPを確認します

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
kubectl get po -o wide
```

```
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          26s   10.42.0.18   k3sarm   <none>           <none>
```

一時的に作成した busybox イメージ から wget でアクセスします

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.42.0.18
```

以下の方法でも同じことができます

```bash
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

</p>
</details>

### PodのYAMLを確認する

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

### Podの詳細と潜在的な問題について情報を取得する

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Podのログを確認する

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### Podが再作成された際に、一つ前のログを確認する

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
# or
kubectl logs nginx --previous
```

</p>
</details>

### nginx podでシェルを実行する

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

</p>
</details>

### busybox pod を作って、そこでecho 'hello world'を実行する

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# or
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### busybox pod を作って、そこでecho 'hello world'を実行し、終わったらPodを終了させる

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### nginx podを環境変数に'var1=val1'をセットしつつ作成し. Podの環境変数を確認する

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
