
## Install OpenWhisk

Install [OpenWhisk](https://openwhisk.apache.org/), a framework for running serverless functions:

Create an OpenWhisk configuration file `owconfig.yaml`:

```yaml
# Single worker node configuration file, so set the node affinity to false
affinity:
  enabled: false

toleration:
  enabled: false

invoker:
  options: "-Dwhisk.kubernetes.user-pod-node-affinity.enabled=false"

whisk:
  ingress:
    type: NodePort
    api_host_name: http://localhost/
    api_host_port: 31001

nginx:
  httpsNodePort: 31001
```

Label OpenWhisk worker nodes (assuming a single worker node in this case): 

```bash
k3s kubectl label nodes --all openwhisk-role=invoker --kubeconfig /etc/rancher/k3s/k3s.yaml
#... '/etc/rancher/k3s/k3s.yaml' is the default k3s kubeconfig filepath
#... Substitute with your own path if it is different 
```

Install OpenWhisk through helm charts:

```bash
helm repo add openwhisk https://openwhisk.apache.org/charts
helm repo update
helm install owdev openwhisk/openwhisk -n openwhisk --create-namespace -f owconfig.yaml --kubeconfig /etc/rancher/k3s/k3s.yaml
#... '/etc/rancher/k3s/k3s.yaml' is the default k3s kubeconfig filepath
#... Substitute with your own path if it is different 
```

Check that all pods are getting deployed and run properly:

```bash
k3s kubectl get pod -n openwhisk -w
#... This can take a few minutes
```

## Install OpenWhisk CLI


Install the [OpenWhisk CLI](https://github.com/apache/openwhisk-cli):

```bash
# For example for release 1.2.0:
wget https://github.com/apache/openwhisk-cli/releases/download/1.2.0/OpenWhisk_CLI-1.2.0-linux-amd64.tgz && \
tar -xvf OpenWhisk_CLI-1.2.0-linux-amd64.tgz && \
sudo mv wsk /usr/local/bin/
```

Make sure the CLI tool was installed correctly by running:

```bash
wsk --help
```

Set CLI properties:  
 
```bash
# Set value <host>:<port>
# NOTE: These values should correspond with whatever was set in the 'owconfig.file' described above
wsk property set --apihost localhost:31001

# Set an authorization key (username and password) that grants access to the OpenWhisk API
# NOTE: The guest auth key is the default value defined here:
# --> https://github.com/apache/openwhisk/blob/master/ansible/files/auth.guest 
wsk property set --auth 23bc46b1-71f6-4ed5-8c54-816aa4f8c502:123zO3xZCLrMN6v2BKK1dXYFpXlPkccOFqm12CdAsMgRU4VrNZ9lyGVCGuMDGIwP
```

## Deploy & run a JavaScript test function 

Create a file `hello.js`:

```js
/**
 * "Hello" OpenWhisk action (serverless function) 
 */
function main({name}) {
    var msg = 'You did not tell me who you are...';
    if (name) {
        msg = `Hello ${name}!`
    }
    return {body: `<html><body><h3>${msg}</h3></body></html>`}
}
```

Create a package and a serverless funtion:

```bash
wsk -i package create demo && \
wsk -i action create /guest/demo/hello hello.js --web true
```

Trigger the serverless function:

```bash
# A: By invoking the function from the commandline:
wsk -i action invoke /guest/demo/hello --result --param name YourNameHere

# B: By retrieving and visiting the URL that triggers the function:
# * Retrieve URL
wsk -i action get /guest/demo/hello --url

# * Visit URL through a browser or curl
curl -k -d '{"name": "YourNameHere"}' -H "Content-Type: application/json" -X POST https://localhost:31001/api/v1/web/guest/demo/hello; echo
#... Use flag '-k' to accept self-signed certificates
```

For more information on how GET, POST, PUT and DELETE methods work see the [OpenWhisk API gateway docs](https://github.com/apache/openwhisk/blob/master/docs/apigateway.md)