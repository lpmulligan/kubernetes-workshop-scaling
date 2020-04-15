# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial:  [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).


## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

### Using the Azure CLI

You can install kubectl directly from the `az` command line tool.

```shell
az aks install-cli
```

### OS X

```shell
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
```

```shell
chmod +x ./kubectl
```

```shell
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Linux

```shell
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

```shell
chmod +x ./kubectl
```

```shell
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Windows
Note you need to have chocolately package manager installed first (https://chocolatey.org/)

```shell
PS C:\Windows\system32>choco install kubernetes-cli
```

### Verification

Verify `kubectl` version 1.17.0 or higher is installed:

```shell
kubectl version --client
```

> output

```shell
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.3", GitCommit:"06ad960bfd03b39c8310aaf92d1e7c12ce618213", GitTreeState:"clean", BuildDate:"2020-02-13T18:08:14Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"darwin/amd64"}
```

To quick check kubectl version, you can also use the following command : 

```shell
kubectl version --short
```

> output

```shell
Client Version: v1.17.3
```

Next: [Resource Limits and QoS](03-resource-limits-and-qos.md)