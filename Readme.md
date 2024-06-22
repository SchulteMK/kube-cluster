# my home k3s cluster

## cluster deployment
### k3s install

download and install k3s:
```
curl -sfL https://get.k3s.io | sh -s - server --write-kubeconfig-mode 644 --cluster-cidr=fd42::/48 --service-cidr=fd43::/112 --disable=servicelb,traefik,local-storage,metrics-server --flannel-backend=none --disable-network-policy --node-ip={nodeIP}
```
extract kubeconfig with `cat /etc/rancher/k3s/k3s.yaml` and copy it to local path `.kube/config` on your admin machine. Replace the local ip adress with the actual ip of the node. You should be able to connect to the freshly installed k3s instance and list your first node with `kubectl get nodes`.
The output should look something like this:

![cluster nodes without cni installed](img/nodes-noCNI.png)

### cilium deployment
install cilium cli to your admin machine ([cilium.io](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli)):
```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```
As we are using IPv6 and BGP, install cilium with the following options:

`
cilium install --set ipv6.enabled=true,bgpControlPlane.enabled=true
`

After that you can wait until the deployment is done with `cilium status --wait`. After the deployment finishes, your node should be ready `kubectl get nodes`:
![cluster nodes with cni installed](img/nodes-CNI.png)

You can also take a look at all the deployed pods `kubectl get pods -A`:
![empty cluster after cni installation](img/pods-emptyCluster.png)

Now the k3s cluster with cilium cli is sucessfully deployed.

## Argo CD
[Argo CD](https://argo-cd.readthedocs.io/en/stable/) is a declarative, GitOps continuous delivery tool for Kubernetes. We are going to deploy everything declarative through argocd. 

### Option A
First we will deploy argocd manually, afterwards we will add the argocd deployment as an application to argocd itself. This way argocd will manage itself after its initial deployment.

### Option B
Create argocd application yaml files. Deploy argocd including the application to manage itself.

---
---
---
non structures notes below

---
---
---

## infrastructure apps
### argocd deployment
app of apps
### deploy infrastructure repository
if needed, restore backed up certificates for sealed secrets

### deploy app repository

## ArgoCD repo
1. argocd

## Infrastructure repo
1. sealed secret
1. cert-manager
1. externaldns
1. storage provisioning

## App layer 1 repo
1. authentik

## App layer 2 repo
1. immich
1. paperless
1. nextcloud