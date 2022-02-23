# Procedure
 1.   Install Tanzu Community Edition.
 2.  Install Azure CLI(contain create Azure account) and set up it.
 3. Create Your own management node and work node.
 4. Deploy a sample app on your cluster
 

# Install Tanzu Community Edition

### 1. Install Docker
 [Install Docker 4.2.0(Don't update it, you need to make sure that the cgroup is v1)](https://docs.docker.com/desktop/mac/release-notes/#docker-desktop-420)
Then Run this command to maker sure that  your cgroup is v1.

```bash
docker info | grep -i cgroup
```

### 2. Install Kubectl

```bash
# Intel core
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"

# apple core
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"
```
If it is not work , Run this command:
 

```bash
brew install kubectl 
```

### Install Tanzu Community
#### Option 1: use Homebrew
Run this command:
```bash
brew install vmware-tanzu/tanzu/tanzu-community-edition
```
Then run sh to install it.

```bash
# the default homebrew install location is /usr/local/Cellar/
<HOMEBREW-INSTALL-LOCATION>/v0.9.1/libexec/configure-tce.sh
```
#### Opetion 2: use the binary download
Download it.[link](https://github.com/vmware-tanzu/community-edition/releases/download/v0.9.1/tce-darwin-amd64-v0.9.1.tar.gz)
go to the download dictionary and run install script

```bash
cd <DOWNLOAD-DIR>
tar xzvf ~/<DOWNLOAD-DIR>/tce-darwin-amd64-v0.9.1.tar.gz
cd tce-darwin-amd64-v0.9.1
./install.sh
# if you want to uninstall it run this command in this folder
./uninstall.sh
```
Run this command to check if you have successful install it. ***(Before run this command, you need to make sure that your docker application is opening)***
```bash
# it is located at http://127.0.0.1:8080
tanzu management-cluster create --ui
```
If you can open a page named Welcome to the Tanzu Community Edition Installer.  it means that you have finished this part.

# Install Azure CLI
### 1. Create Azure account and use it to log in to azure portal. 
After this step, please check your subscription plan on this website.[link](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)(if there show noting, you need to wait for a email told you that you account has been approved)

### 2. Install Azure CLI

```bash
brew update && brew install azure-cli
```
if you can not do it, check this [link](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-macos)

###  3. Register Tanzu Community Edition as an Azure Client App.
Follow this instruction[link](https://tanzucommunityedition.io/docs/latest/azure-mgmt/#tkg-app)
(Notice: after you create your Client Secrets, record it because you can only see it once)

### 4. Accept the Base Image License
***Notice:This step is to make sure that you can successfully create an VM on your server, So if you need to create other VM version, you need to accept other License.***

1. Sign in to the Azure CLI as your tce client application.

```bash
az login --service-principal --username AZURE_CLIENT_ID --password AZURE_CLIENT_SECRET --tenant AZURE_TENANT_ID
```
Where AZURE_CLIENT_ID, AZURE_CLIENT_SECRET, and AZURE_TENANT_ID are your tce app’s client ID and secret and your tenant ID, as recorded in Register Tanzu Community Edition as an Azure Client App.

2. agree with the term you want
if you want to use ubuntu 18.04, you need to change 
k8s-1dot21dot2-ubuntu-2004 to k8s-1dot21dot2-ubuntu-1804
```bash
az vm image terms accept --publisher vmware-inc --offer tkg-capi --plan k8s-1dot21dot2-ubuntu-2004 --subscription AZURE_SUBSCRIPTION_ID
```
Check it in your Azure portal (Subscription--> your subscription -> Programmatic deployment)


# Create Your own management node and work node
Follow the instruction [here](https://tanzucommunityedition.io/docs/latest/azure-install-mgmt/). And for this part, I will notice some normal bugs and fix it.

### failed in the ***Setup bootstrap cluster*** or ***install providers on bootstrap cluster***
If you failed in the ***Setup bootstrap cluster*** or ***install providers on bootstrap cluster***, it means that your local cluster need to clean up. follow the steps here to completely clean it up.
1. clean your cluster on your local machine and azure server. (after do this, if you still failed, then run step 2)

```bash
tanzu management-cluster delete
```

2. clean up your configure files
```bash
# nanohana is my user name, pls use your user namer here
cd /Users/nanohana/.config/tanzu/tkg/clusterconfigs
rm -f *
cd /Users/nanohana/.kube-tkg/tmp
rm -f *
```
3. clean up your docker
open the docker application and remove the docker we used for our management cluster.

### failed in the ***Create management cluster***
You can check the reason by use azure portal(resources group--> your resources -> activity log)

For most situation, the bug is you do not accept the right term or you exceed your core number quota.

for the first one, follow the instruction in the activity log.
for the second one, please use less core to create your management node and work node.

### After you create cluster, check it in your terminal
Run this command.
```bash
tanzu management-cluster get
```
if you result is like this, you are succeed.
![在这里插入图片描述](https://img-blog.csdnimg.cn/b49965b186024e7983def30325cbb2d2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA55WM6ZmQ5LiN5a2Y5Zyo55qE,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
#  Deploy a sample app on your cluster
Install octant
```bash
brew install octant
# check it
octant
```
### deploy the kuard app

```bash
# Deploy the kuard demo app.
kubectl run --restart=Never --image=gcr.io/kuar-demo/kuard-amd64:blue kuard

# Verify that the pod is running.
kubectl get pods

# Forward the pod container port 8080 to your local host port 8080.
kubectl port-forward kuard 8080:8080
# go to http://localhost:8080.
# Delete the kuard pod.
kubectl delete pod kuard
# Verify that the pod is deleted.
kubectl get pods
```

