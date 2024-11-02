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
kind create cluster --name helm --image kindest/node:v1.31.1
```

## Install Helm CLI

```
curl -LO https://get.helm.sh/helm-v3.4.0-linux-amd64.tar.gz
tar -C /tmp/ -zxvf helm-v3.4.0-linux-amd64.tar.gz
rm helm-v3.4.0-linux-amd64.tar.gz
mv /tmp/linux-amd64/helm /usr/local/bin/helm
chmod +x /usr/local/bin/helm

```

cd helm
mkdir temp && cd temp

kubectl get nodes

helm tamplete [Name] [CHART folder] - present us the yaml of all the chart we have and if something worng we will get an error.

helm install [Name] [CHART folder]

example:

NAME: exapmle-app
LAST DEPLOYED: Fri Nov 1 09:48:07 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

helm list (present the release that run in the cluster)
example:
NAME NAMESPACE REVISION UPDATED STATUS CHART APP VERSION
example-app default 1 2024-11-02 22:10:25.731007268 +0000 UTC deployed exapmle-app-0.1.0 1.16.0

## Value injections for our Chart

For CI systems, we may want to inject an image tag as a build number <br/>

Basic parameter injection: <br/>

```
# values.yaml

deployment:
  image: "aimvector/python"
  tag: "1.0.4"

# deployment.yaml

image: {{ .Values.deployment.image }}:{{ .Values.deployment.tag }}

# upgrade our release

helm upgrade [Name] [Chart directory] --values [Path to Values file]

example:
helm upgrade example-app example-app --values ./example-app/values.yaml

NAME: example-app
LAST DEPLOYED: Sat Nov  2 22:18:51 2024
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None

We can see that the REVISION is now 2

# see revision increased

helm list

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
example-app     default         2               2024-11-02 22:18:51.476648288 +0000 UTC deployed        exapmle-app-0.1.0       1.16.0

For Secret we can use the --set Flag instead --values Like this:

helm upgrade example-app example-app --set deployment.tag=1.0.4
```

we can copy the values file and like values-2.yaml and give diff name for another app Like:

helm install example-app-2 example-app --values ./example-app/example-app2.values.yaml

NAME: example-app-2
LAST DEPLOYED: Sat Nov 2 22:43:10 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

helm list

```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
example-app     default         3               2024-11-02 22:55:57.64759638 +0000 UTC  deployed        exapmle-app-0.1.0       1.16.0
example-app-2   default         3               2024-11-02 22:55:14.720495698 +0000 UTC deployed        exapmle-app-0.1.0       1.16.0
```

## If\Else and Default values

You can also set default values in case they are not supplied by the `values.yaml` file. <br/>
This may help you keep the `values.yaml` file small <br/>

```
{{- if .Values.deployment.resources }}
  resources:
    {{- if .Values.deployment.resources.requests }}
    requests:
      memory: {{ .Values.deployment.resources.requests.memory | default "50Mi" | quote }}
      cpu: {{ .Values.deployment.resources.requests.cpu | default "10m" | quote }}
    {{- else}}
    requests:
      memory: "50Mi"
      cpu: "10m"
    {{- end}}
    {{- if .Values.deployment.resources.limits }}
    limits:
      memory: {{ .Values.deployment.resources.limits.memory | default "1024Mi" | quote }}
      cpu: {{ .Values.deployment.resources.limits.cpu | default "1" | quote }}
    {{- else}}
    limits:
      memory: "1024Mi"
      cpu: "1"
    {{- end }}
  {{- else }}
  resources:
    requests:
      memory: "50Mi"
      cpu: "10m"
    limits:
      memory: "1024Mi"
      cpu: "1"
  {{- end}}


# rollout the change

helm upgrade example-app example-app --values ./example-app/example-app-01.values.yaml
```
