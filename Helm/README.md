## Get a container to work in

<br/>
Run a small `alpine linux` container where we can install and play with `helm`: <br/>

```
docker run -it --rm -v ${HOME}:/root/ -v ${PWD}:/work -w /work --net host alpine sh

# install curl & kubectl
apk add --no-cache curl nano
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
export KUBE_EDITOR="nano"

# test cluster access:
/work # kubectl get nodes
NAME                    STATUS   ROLES    AGE   VERSION
helm-control-plane   Ready    master   26m   v1.26.0

```

# test cluster access:

/work # kubectl get nodes
NAME STATUS ROLES AGE VERSION
helm-control-plane Ready master 26m v1.26.0

cd kubernetes/daemonsets

## We need a Kubernetes cluster

Lets create a Kubernetes cluster to play with using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

```
kind create cluster --name helm --image kindest/node:v1.26.0
```

kubectl get nodes
