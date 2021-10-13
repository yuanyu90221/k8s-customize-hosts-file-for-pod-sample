# k8s-customize-hosts-file-for-pod-sample

參考文件 https://kubernetes.io/docs/tasks/network/customize-hosts-file-for-pods/

## 前言

今天要來實作 [Adding entries to Pod /etc/hosts with HostAliases](https://kubernetes.io/docs/tasks/network/customize-hosts-file-for-pods/) 這個任務

當 DNS 無沒有辦法轉換時, 在 Pod 的 /etc/hosts 新增欄位能夠做到 Pod 級別的域名轉換

可以在 PodSpec 的 HostAlias 欄位新增要轉換的域名來達到上述的要求

/etc/hosts 主要是被 kubelet 所管理, 當 Pod 產生或是重啟時, 這個檔案的設定都有可能被複寫掉, 所以不建議用 HostAlias 的改法

## 佈署目標


1 建立一個 nginx 的 Pod, 並檢查原始 nginx container 內的 /etc/hosts

2 新增一個 Pod 使用 HostAlias 新增域名轉換並且透過 cat /etc/hosts 把值印出來


## 建立一個 nginx 的 Pod, 並檢查原始 nginx container 內的 /etc/hosts

建立 nginx Pod 使用以下指令

```shell=
kubectl run nginx --image nginx
```
![](https://i.imgur.com/QHLZ41D.png)

查看 Pod IP 使用以下指令

```shell=
kubectl get pods --output=wide
```

![](https://i.imgur.com/0Sm1pPY.png)

查看 container 內的 /etc/hosts 檔案使用以下指令

```shell=
kubectl exec nginx -- cat /etc/hosts
```
![](https://i.imgur.com/hYgO5HG.png)

## 新增一個 Pod 使用 HostAlias 新增域名轉換並且透過 cat /etc/hosts 把值印出來

建立 hostaliases-pod.yaml 如下

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: hostaliases-pod
spec:
  restartPolicy: Never
  hostAliases:
  - ip: "127.0.0.1"
    hostnames:
    - "foo.local"
    - "bar.local"
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
  containers:
  - name: cat-hosts
    image: busybox
    command:
    - cat
    args:
    - "/etc/hosts"
```

建立一個 Pod 

名稱設定為 hostaliases-pod

設定 container image 使用 busybox

設定 container 執行指令 如下

```shell=
cat /etc/hosts
```

設定 hostAliases 如下

```yaml=
hostAliases:
- ip: "127.0.0.1"
  hostnames:
  - "foo.local"
  - "bar.local"
- ip: "10.1.2.3"
  hostnames:
  - "foo.remote"
  - "bar.remote"
```

建立佈署使用以下指令

```shell=
kubectl apply -f hostaliases-pod.yaml
```
![](https://i.imgur.com/KOt57Xl.png)

查看 Pod 的 IP 使用以下指令

```shell=
kubectl get pod --output=wide
```
![](https://i.imgur.com/piZQG5L.png)

查看 Pod 的 log 使用以下指令

```shell=
kubectl logs hostaliases-pod
```

![](https://i.imgur.com/ElJN28A.png)

## 後記

為何用 kubelet 管理 hosts 檔案？

主要是為了避免 Container 在啟動後去做修改

因為如果在 Container 做修改, 這些修改都會在 Container 重起之後消失