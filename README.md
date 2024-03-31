## K3s Rancher Installation Commands for RHEL

### Install K3s and Initiate a Cluster
``` bash
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.27.10+k3s2 sh -s - server --etcd-expose-metrics true --cluster-domain rancher-cluster.local --node-name master0 --cluster-init
```

### Installing HELM for Later Use
``` bash
sudo dnf install wget -y
sudo wget https://get.helm.sh/helm-v3.14.2-linux-amd64.tar.gz
sudo dnf install tar
sudo tar -zxvf helm-v3.14.2-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

### Setting Up Path for KUBECONFIG
This step is to aovid 'cluster unreachable' warning.
``` bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

### Applying Cert Manager 
``` bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.3/cert-manager.crds.yaml
```

### Creating `cattle-system` Namespace
This namespace will be used to create Rancher-related pods.
``` bash
kubectl create namespace cattle-system
```

### Update Helm Repo To Install Rancher Server and Jetstack/cert-manager
``` bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

### Installing and Checking Cert-Manager
Rancher uses cert-manager to generate self-signed certificates.
If the second command returns running container info, the cert-manager is up. Make sure all the containers are running and ready.
``` bash
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace
kubectl get pods --namespace cert-manager
```

### Installing Rancher on K3s Using Helm
> Please do not add angle brackets in the actual command.
``` bash
helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=<your-ip-address>.sslip.io   --set bootstrapPassword=<password>   --set useBundledSystemChart=true   --set replicas=1 --set ingress.tls.source=rancher
```

### Checking Status
The following command retrieves all the containers and their state in the `cattle-system` namespace. Make sure all the containers (rancher and rancher webhook) are running.
```bash
kubectl get all -n cattle-system
```
Now please use the host IP address provided in the command to launch Rancher. Use username 'admin' and bootstrap password for initial login.


## Troubleshooting
### Bad Gateway
If there is `bad gateway` message while trying to reach rancher, please consider disabling the firewall on the host using following commands:
``` bash
systemctl stop firewalld.service
systemctl disable firewalld
```
