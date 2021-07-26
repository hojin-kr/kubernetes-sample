# kubernetes-sample
kubernetes sample
고가용성 분사 처리 서버 시스템 Kubernetes

## 목적
개발에서 라이브까지 Kubernetes 서버 시스템 배포 샘플

## 과정
1. sets up a local Kubernetes 
   1. minikube
   2. Writing Deployment.yaml
   3. 배포및 스케일링
   4. 추가 배포및 배포간 전환
   5. 지속 배포 (github Action)

2. 상용 클라우드 시스템에서 배포
   1. GOOGLE GKE
      1. test
   3. AWS EKS

## sets up a local Kubernetes
### [minikube](https://minikube.sigs.k8s.io/docs/)
#### Running [minikube](https://minikube.sigs.k8s.io/docs/)
[minikube](https://minikube.sigs.k8s.io/docs/) sets up a local Kubernetes cluster 
```
➜  ~ brew install minikube
Updating Homebrew...
```
```
➜  ~ minikube start
😄  minikube v1.22.0 on Darwin 10.15.7
✨  Automatically selected the docker driver. Other choices: hyperkit, virtualbox, ssh
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.21.2 preload ...
```
#### Checking kubernetes Cluster & Nodes
```
➜  ~ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4m37s
```
Or
```
➜  ~ minikube dashboard
🔌  Enabling dashboard ...
    ▪ Using image kubernetesui/dashboard:v2.1.0
    ▪ Using image kubernetesui/metrics-scraper:v1.0.4
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:57142/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...

```
### Writing Deployment.yaml 
