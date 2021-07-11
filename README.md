# Overview

This repository contains a small collection of documents with instructions and links to helpful external resource. The instructions and links have been collected for the purpose of easily setting up a Kubernets cluster for running serverless functions using the [OpenFaaS](https://www.openfaas.com/) or [OpenWhisk](https://openwhisk.apache.org/) framework. 


## Install a Kubernetes Cluster (k3s) 

Install [k3s](https://k3s.io/), a small and lightweight Kubernetes distribution:

```bash 
curl -sfL https://get.k3s.io | sh -
```

Check that everything was installed properly by seeing whether or not a node is actually listed:

```bash 
k3s kubectl get node
```

## Install Helm

Install [helm](https://helm.sh/), a Kubernetes package manager:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
rm -f get_helm.sh
```

## Install OpenFaaS

For OpenFaaS installation instructions see [here](./OpenFaaS.md). For more information visit the [OpenFaaS docs](https://docs.openfaas.com/).

## Install OpenWhisk

For OpenWhisk installation instructions see [here](./OpenWhisk.md). For more information visit the [OpenWhisk docs](https://openwhisk.apache.org/documentation.html).

## (Optional) Install Sealed Secrets 

SealedSecret can be used to encrypt secrets. A "sealed secret" can be decrypted only by the controller running in the target cluster and nobody else. In the instructions below it is assumed that cluster/server-side resources are deployed to a k3s kubernetes cluster.

### Client-side setup & configuration

#### Install sealed secrets client-side 

Install the latest version (0.16.0 at the time of writing) of the sealed secrets client-side program:

```bash
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.16.0/kubeseal-linux-amd64 -O kubeseal && \
sudo install -m 755 kubeseal /usr/local/bin/kubeseal && \
rm -f kubeseal
```

To get a more recent version visit the relevant github repository [here](https://github.com/bitnami-labs/sealed-secrets/releases).

### Cluster/server-side setup & configuration

(Optional) create a sealed secrets namespace:

```bash
sudo k3s kubectl create namespace sealed-secrets
```

Download manifests for the sealed secrets controller:

```bash
curl -L kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.16.0/controller.yaml > sealed-secrets.yaml
```

Again, to get a more recent version visit the relevant github repository [here](https://github.com/bitnami-labs/sealed-secrets/releases).

If another `sealed-secrets` namespace was created then change the default namespace of the sealed secrets controller from `kube-system` to `sealed-secrets`. Otherwise the controller will just be deployed to the default `kube-system` namespace.

```bash
sed -i -e "s/kube-system/sealed-secrets/g" sealed-secrets.yaml
```

Finally, deploy the sealed secrets controller:

```bash
sudo k3s kubectl apply -f sealed-secrets.yaml && \
rm -rf sealed-secrets.yaml
```

Verify that a sealed secrets pod has been deployed and is up and running:

```bash
sudo k3s kubectl get pods --namespace sealed-secrets
```