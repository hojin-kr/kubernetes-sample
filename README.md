# kubernetes-sample
kubernetes sample
고가용성 분사 처리 서버 시스템 Kubernetes

## 목적
개발에서 라이브까지 Kubernetes 서버 시스템 배포 샘플

## 과정
1. sets up a local Kubernetes 
   1. minikube
   2. kubernetes 정의 
   3. 배포및 스케일링
   4. 추가 배포및 배포간 전환
   5. 지속 배포 (github Action)

2. 상용 클라우드 시스템에서 배포
   1. GOOGLE GKE
      1. test
   3. AWS EKS

## sets up a local Kubernetes 
sets up a local Kubernetes cluster 
(minikube)[https://minikube.sigs.k8s.io/docs/]
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
