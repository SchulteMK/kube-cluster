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

Now the k3s cluster with cilium cli is sucessfully deployed. Now would be a great time to create a vm snapshot. If something goes wrong, you always come back to this point in time.

## Argo CD
[Argo CD](https://argo-cd.readthedocs.io/en/stable/) is a declarative, GitOps continuous delivery tool for Kubernetes. We are going to deploy everything declarative through argocd. 

install [argocd cli](https://argo-cd.readthedocs.io/en/stable/cli_installation/#download-with-curl):
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

We are starting with a [default deployment](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd) to get started: 
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

We can not yet expose argocd with an `LoadBalancer`, because our cilium installation cannot route any bgp traffic nor hand out any ips. So just for the initial setup we making use of the port forwarding included in `kubectl`: `kubectl port-forward svc/argocd-server -n argocd 8080:443`. We can retrieve the default admin credentials via `argocd admin initial-password -n argocd`, or just by reading the secret named `argocd-initial-admin-secret`: `kubectl get secret -n argocd argocd-initial-admin-secret --template={{.data.password}} | base64 -d`

You can login through the webinterface (https://localhost:8080/) or through the cli as described [here](https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli). First of we are gonna change the initial admin password and afterwards delete the secret containing it: `kubectl delete secret -n argocd argocd-initial-admin-secret`

### First argocd application


---
non structures notes below
---

### Option A
First we will deploy argocd manually, afterwards we will add the argocd deployment as an application to argocd itself. This way argocd will manage itself after its initial deployment.

### Option B
Create argocd application yaml files. Deploy argocd including the application to manage itself.

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