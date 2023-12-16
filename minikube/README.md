# Minikube on AWS EC2

### Install docker on EC2(t3.medium and Ubuntu 20.04 LTS)
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release

sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl status docker
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu
newgrp docker
sudo systemctl restart docker
docker ps
```

### Install Kubectl
```
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
```
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
mv kubectl /bin/kubectl
chmod a+x /bin/kubectl
```

### Install Minikube
```
https://aws.plainenglish.io/running-kubernetes-using-minikube-cluster-on-the-aws-cloud-4259df916a07
https://minikube.sigs.k8s.io/docs/start/
https://minikube.sigs.k8s.io/docs/drivers/none/#requirements
```
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/bin/minikube
sudo apt install conntrack -y
```
If you want to use the combination of the none driver from Kubernetes v1.24+ the Docker container runtime you'll need to install cri-dockerd on your system
Install cri-dockerd & Install go
### Install cri-dockerd
```
apt install git -y
apt  install golang-go
git clone https://github.com/Mirantis/cri-dockerd.git
```
```
mkdir bin
VERSION=$((git describe --abbrev=0 --tags | sed -e 's/v//') || echo $(cat VERSION)-$(git log -1 --pretty='%h')) PRERELEASE=$(grep -q dev <<< "${VERSION}" && echo "pre" || echo "") REVISION=$(git log -1 --pretty='%h')
go build -ldflags="-X github.com/Mirantis/cri-dockerd/version.Version='$VERSION}' -X github.com/Mirantis/cri-dockerd/version.PreRelease='$PRERELEASE' -X github.com/Mirantis/cri-dockerd/version.BuildTime='$BUILD_DATE' -X github.com/Mirantis/cri-dockerd/version.GitCommit='$REVISION'" -o cri-dockerd
```
```
# Run these commands as root
###Install GO###
wget https://storage.googleapis.com/golang/getgo/installer_linux
chmod +x ./installer_linux
./installer_linux
source ~/.bash_profile
```
```
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```

### Install CRICTL
https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md
```
VERSION="v1.26.0" # check latest version in /releases page
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz

cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false
EOF
```
# How do I install containernetworking-plugins for none driver?
Pick the version from here -> https://github.com/containernetworking/plugins/releases
```
CNI_PLUGIN_VERSION="<version_here>"
CNI_PLUGIN_TAR="cni-plugins-linux-amd64-$CNI_PLUGIN_VERSION.tgz" # change arch if not on amd64
CNI_PLUGIN_INSTALL_DIR="/opt/cni/bin"

curl -LO "https://github.com/containernetworking/plugins/releases/download/$CNI_PLUGIN_VERSION/$CNI_PLUGIN_TAR"
sudo mkdir -p "$CNI_PLUGIN_INSTALL_DIR"
sudo tar -xf "$CNI_PLUGIN_TAR" -C "$CNI_PLUGIN_INSTALL_DIR"
rm "$CNI_PLUGIN_TAR"
```
### Start Minikube
```
minikube start --vm-driver=none
```

### Test the setup
```
kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
kubectl expose pod hello-minikube --type=NodePort

```

# minikube anb kubectl commands

# Minikube Commands:
```
Start Minikube: minikube start
Stop Minikube: minikube stop
Delete Minikube: minikube delete
View Minikube status: minikube status
Enable Minikube dashboard: minikube dashboard
SSH into Minikube: minikube ssh
Get Minikube IP: minikube ip
View Minikube configuration: minikube config view
Enable Minikube addons: minikube addons enable ADDON_NAME
Disable Minikube addons: minikube addons disable ADDON_NAME
Pause Minikube: minikube pause
Unpause Minikube: minikube unpause
Enable Minikube metrics server: minikube addons enable metrics-server
Enable Minikube Ingress addon: minikube addons enable ingress
Disable Minikube Ingress addon: minikube addons disable ingress
Open Minikube service in default browser: minikube service SERVICE_NAME
Pause Minikube Kubernetes components: minikube pause
Unpause Minikube Kubernetes components: minikube unpause
Upgrade Minikube: minikube upgrade
Set Minikube VM driver: minikube config set driver DRIVER_NAME
View Minikube addons: minikube addons list
SSH into a specific node: minikube ssh -n NODE_NAME
Change Minikube VM memory: minikube config set memory MEMORY_AMOUNT
Change Minikube VM CPU: minikube config set cpus CPU_COUNT
Start Minikube with a specific profile: minikube start -p PROFILE_NAME
```
# Kubectl Commands:
```
Get pods: kubectl get pods
Get services: kubectl get services
Get deployments: kubectl get deployments
Describe a resource: kubectl describe RESOURCE_NAME RESOURCE_TYPE
Create a resource from a YAML file: kubectl apply -f FILE_NAME
Delete a resource using a YAML file: kubectl delete -f FILE_NAME
Scale a deployment: kubectl scale deployment DEPLOYMENT_NAME --replicas=REPLICA_COUNT
Expose a service: kubectl expose deployment DEPLOYMENT_NAME --type=SERVICE_TYPE --port=PORT_NUMBER
Port forward to a pod: kubectl port-forward POD_NAME LOCAL_PORT:POD_PORT
Execute a command in a pod: kubectl exec -it POD_NAME -- COMMAND
View pod logs: kubectl logs POD_NAME
View pod events: kubectl get events
View Kubernetes cluster information: kubectl cluster-info
Create a namespace: kubectl create namespace NAMESPACE_NAME
Set the current context: kubectl config use-context CONTEXT_NAME
View available contexts: kubectl config get-contexts
View the current context: kubectl config current-context
Delete a resource: kubectl delete RESOURCE_TYPE RESOURCE_NAME
View resource YAML definition: kubectl get RESOURCE_TYPE RESOURCE_NAME -o yaml
Execute a command in a specific container of a pod: kubectl exec -it POD_NAME -c CONTAINER_NAME -- COMMAND
View pod details along with labels: kubectl get pods -L LABEL_NAME
View detailed information about a deployment: kubectl describe deployment DEPLOYMENT_NAME
View detailed information about a service: kubectl describe service SERVICE_NAME
Create a secret from a literal value: kubectl create secret generic SECRET_NAME --from-literal=KEY=VALUE
Create a secret from a file: kubectl create secret generic SECRET_NAME --from-file=PATH_TO_FILE
View secrets in a specific namespace: kubectl get secrets -n NAMESPACE_NAME
View pod logs from a specific container: kubectl logs POD_NAME -c CONTAINER_NAME
Expose a pod as a service: kubectl expose pod POD_NAME --port=SERVICE_PORT --target-port=POD_PORT
View PersistentVolume information: kubectl get pv
View PersistentVolumeClaim information: kubectl get pvc
Describe PersistentVolume: kubectl describe pv PV_NAME
Describe PersistentVolumeClaim: kubectl describe pvc PVC_NAME
```

