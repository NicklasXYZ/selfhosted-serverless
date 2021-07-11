## Install OpenFaaS 

Install [OpenFaaS](https://www.openfaas.com/), a framework for running serverless functions:

```bash
# Deploy namespaces
k3s kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml

# Install OpenFaaS in the cluster
#... '/etc/rancher/k3s/k3s.yaml' is the default k3s kubeconfig filepath
#... Substitute with your own path if it is different 
helm repo add openfaas https://openfaas.github.io/faas-netes/ --kubeconfig=/etc/rancher/k3s/k3s.yaml 
helm repo update --kubeconfig=/etc/rancher/k3s/k3s.yaml
helm upgrade --kubeconfig=/etc/rancher/k3s/k3s.yaml openfaas --install openfaas/openfaas --namespace openfaas --set functionNamespace=openfaas-fn --set generateBasicAuth=true
```

Retrieve OpenFaaS password (needed below, but also when the public user interface, usually running at http://localhost:31112/ui/, need to be accessed):

```bash
PASSWORD=$(sudo k3s kubectl -n openfaas get secret basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode) && \
echo "OpenFaaS admin password: $PASSWORD"
```

## Install OpenFaaS CLI

Install the OpenFaaS CLI:

```bash
curl -sSL https://cli.openfaas.com | sudo sh
```

Authenticate with OpenFaaS (replace the gateway address if it is different):

```bash 
faas-cli login -u admin --password $PASSWORD --gateway http://localhost:31112
```

Deploy the "nodeinfo" test function:

```bash
faas-cli deploy --image NodeInfo --name nodeinfo --gateway http://localhost:31112
```

This "nodeinfo" function can be reached on the internal address: http://gateway.openfaas:8080 (using the pattern `https://<service name>.<namespace>:<default port>`) or through the public interface, usually running at `http://localhost:31112/ui`, using username `admin` and the previously defined password contained in environment variable `$PASSWORD`.
