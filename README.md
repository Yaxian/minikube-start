# Run minikube

## Run minikube in macOS(10.13.6)

### install minikube

Please follow this [tutorial](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-kubectl)

```
$ minikube version
```
  > minikube version: v0.33.1

### install vm driver

I used [hyperkit](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperkit-driver)

### 1. Prepare docker images dependended and Start minikube

Run `docker pull` the following images on `Host` terminal:

```
	k8s.gcr.io/kube-apiserver:v1.13.2
	k8s.gcr.io/kube-controller-manager:v1.13.2
	k8s.gcr.io/kube-scheduler:v1.13.2
	k8s.gcr.io/kube-proxy:v1.13.2
	k8s.gcr.io/pause:3.1
	k8s.gcr.io/etcd:3.2.24
	k8s.gcr.io/coredns:1.2.6
```

Save the images as `k8s.zip`:
```
$ docker save \
	k8s.gcr.io/kube-apiserver:v1.13.2 \
	k8s.gcr.io/kube-controller-manager:v1.13.2 \
	k8s.gcr.io/kube-scheduler:v1.13.2 \
	k8s.gcr.io/kube-proxy:v1.13.2 \
	k8s.gcr.io/pause:3.1 \
	k8s.gcr.io/etcd:3.2.24 \
	k8s.gcr.io/coredns:1.2.6 \
	| gzip -c > k8s.zip
```

After then, run the following commands:
```
$ sudo rm /var/db/dhcpd_leases # before you remove it, try 'cat /var/db/dhcpd_leases'
$ minikube stop
$ minikube delete
$ minikube start --vm-driver=hyperkit -v=9
```

### 1.1 fail to download ios

Download [minikube-v0.33.1.iso](//storage.googleapis.com/minikube/iso/minikube-v0.33.1.iso) and move it into `~/.minikube/iso/` path.


### 1.2 hang on downloading `kubeadm` and `kubectl`

Retry `minikube start --vm-driver=hyperkit -v=9` some times.

<!-- ### 1.2 fail to download kubeadm and kubelet

* Download [kubeadm](https://storage.googleapis.com/kubernetes-release/release/v1.13.2/bin/linux/amd64/kubeadm)
  * `$ scp kubeadm docker@$(minikube ip):~/`
  * `$ minikube ssh`
  * `$ sudo mv kubeadm /usr/bin/`
  * `$ chmod +x /usr/bin/kubeadm`

* Download [kubelet](https://storage.googleapis.com/kubernetes-release/release/v1.13.2/bin/linux/amd64/kubelet)
  * `$ scp kubelet docker@$(minikube ip):~/`
  * `$ minikube ssh`
  * `$ sudo mv kubelet /usr/bin/`
    * `$ chmod +x /usr/bin/kubelet` -->


### 2. When terminal prints `Starting cluster components...`
We will start a simple HTTP server in `Host` by:

```
$ python -m SimpleHTTPServer
```

to serve `k8s.zip` which will be downloaded in `vm`.

### 3. Login the `vm`

```
$ ssh docker@$(minikube ip) # using password "tcuser"
```

The `/mnt/vda1` has enough space to save `k8s.zip`, run the following commands:

```
$ cd /mnt/vda1
$ sudo wget 'http://host_ip:8000/k8s.zip' # (in my case host ip was 192.168.64.3 but you will have to connect to 192.168.64.1)
$ docker load < k8s.zip
$ sudo rm k8s.zip
$ docker images
```

* If meet the error: `starting cluster:  kubeadm init error`, run following command in `vm`:
```
$ sudo kubeadm reset
$ sudo /usr/bin/kubeadm init --config /var/lib/kubeadm.yaml --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests --ignore-preflight-errors=DirAvailable--data-minikube --ignore-preflight-errors=Port-10250 --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml --ignore-preflight-errors=FileAvailable--etc-kubernetes-manifests-etcd.yaml --ignore-preflight-errors=Swap --ignore-preflight-errors=CRI

```

<!-- ```
$ sudo /usr/bin/kubeadm alpha phase addon kube-dns
``` -->

* Check the log in `vm`
```
$ sudo journalctl -xeu kubelet
```

### 4. Check k8s node

```
$ kubectl version
```

will display log in termina:

```
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.3", GitCommit:"721bfa751924da8d1680787490c54b9179b1fed0", GitTreeState:"clean", BuildDate:"2019-02-04T04:48:03Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.2", GitCommit:"cff46ab41ff0bb44d8584413b598ad8360ec1def", GitTreeState:"clean", BuildDate:"2019-01-10T23:28:14Z", GoVersion:"go1.11.4", Compiler:"gc", Platform:"linux/amd64"}
```

and

```
$ kubectl get nodes
```

will display log in termina:

```
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     master    17m       v1.13.2
```

Have fun with k8s!
