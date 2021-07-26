# kubernetes-sample
kubernetes sample
고가용성 분사 처리 서버 시스템 Kubernetes

## 목적
개발에서 라이브까지 Kubernetes 서버 시스템 배포 샘플

## 과정
1. sets up a local Kubernetes 
   1. minikube
   2. Writing Deployment.yaml
   3. 배포
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
### Writing Deployment
Namespace - Deployment - Service - Ingress

![image](https://user-images.githubusercontent.com/22079767/127005978-500172a5-c646-4ece-96a0-479ca243e329.png)
https://kubernetes.io/docs/concepts/services-networking/ingress/

#### Namespace.yaml
Deploy by defining namespace to manage multiple deployments
```
➜  kubetuto kubectl apply -f namespace.yaml 
namespace/app1 created
➜  kubetuto kubectl get namespace          
NAME                   STATUS   AGE
app1                   Active   5s
default                Active   3h30m
kube-node-lease        Active   3h30m
kube-public            Active   3h30m
kube-system            Active   3h30m
kubernetes-dashboard   Active   3h29m
```
```
apiVersion: v1
kind: Namespace
metadata:
  name: app1
```
### #Deployment.yaml
```
➜  kubetuto kubectl apply -f deployment.yaml
deployment.apps/deployment created
➜  kubetuto kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3h32m
➜  kubetuto kubectl get all --namespace app1
NAME                              READY   STATUS              RESTARTS   AGE
pod/deployment-5f5d7579f9-gfwv4   0/1     ContainerCreating   0          25s
pod/deployment-5f5d7579f9-rtkb6   0/1     ContainerCreating   0          25s
pod/deployment-5f5d7579f9-tn2th   0/1     ContainerCreating   0          25s
pod/deployment-5f5d7579f9-tskj2   0/1     ContainerCreating   0          25s
pod/deployment-5f5d7579f9-xbk4p   0/1     ContainerCreating   0          25s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment   0/5     5            0           26s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-5f5d7579f9   5         5         0       26s
```
```
# Deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  # deployment unique name
  name: deployment
  namespace: app1
spec:
  # Used for identification in other resources
  selector:
    matchLabels:
      app: app1
  # Define the number of pod replicase
  replicas: 5
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
#### Service.yaml
```
➜  kubetuto kubectl apply -f service.yaml 
service/app1-service created
➜  kubetuto kubectl get all --namespace app1
NAME                              READY   STATUS    RESTARTS   AGE
pod/deployment-5f5d7579f9-gfwv4   1/1     Running   0          3m16s
pod/deployment-5f5d7579f9-rtkb6   1/1     Running   0          3m16s
pod/deployment-5f5d7579f9-tn2th   1/1     Running   0          3m16s
pod/deployment-5f5d7579f9-tskj2   1/1     Running   0          3m16s
pod/deployment-5f5d7579f9-xbk4p   1/1     Running   0          3m16s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/app1-service   ClusterIP   10.98.16.238   <none>        80/TCP    7s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment   5/5     5            5           3m16s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-5f5d7579f9   5         5         5       3m16s
➜  kubetuto kubectl get service --namespace app1
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
app1-service   ClusterIP   10.98.16.238   <none>        80/TCP    18s
```
```
apiVersion: v1
kind: Service
metadata:
  name: app1-service
  namespace: app1
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
```
### minikube local에서 service로 노출
로컬에서 minikube로 테스트시 DNS 연결 및 Ingress 테스팅이 어렵기 때문에 minikube의 service 기능으로 service르 로컬에 노출시켜 deploeyment의 Pod에 접근 테스트가 가능하다
```
➜  kubetuto minikube service app1-service --namespace app1
|-----------|--------------|-------------|--------------|
| NAMESPACE |     NAME     | TARGET PORT |     URL      |
|-----------|--------------|-------------|--------------|
| app1      | app1-service |             | No node port |
|-----------|--------------|-------------|--------------|
😿  service app1/app1-service has no node port
🏃  Starting tunnel for service app1-service.
|-----------|--------------|-------------|------------------------|
| NAMESPACE |     NAME     | TARGET PORT |          URL           |
|-----------|--------------|-------------|------------------------|
| app1      | app1-service |             | http://127.0.0.1:59207 |
|-----------|--------------|-------------|------------------------|
🎉  Opening service app1/app1-service in default browser...
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```
#### sercret.yaml
for tls ingress 
https://kubernetes.io/ko/docs/concepts/configuration/secret/
```
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls-ingress
type: kubernetes.io/tls
data:
  tls.crt: |

  tls.key: |

```
#### Ingress.yaml
```
➜  kubetuto kubectl describe ingress app1-tls-ingress --namespace app1
Name:             app1-tls-ingress
Namespace:        app1
Address:          
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
TLS:
  secret-tls-ingress terminates hojintest.shop
Rules:
  Host            Path  Backends
  ----            ----  --------
  hojintest.shop  
                  /   app1-service:80 (172.17.0.5:80,172.17.0.6:80,172.17.0.7:80 + 2 more...)
Annotations:      <none>
Events:           <none>
```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app1-tls-ingress
  namespace: app1
spec:
  tls:
  - hosts:
      - hojintest.shop
    secretName: secret-tls-ingress
  rules:
  - host: hojintest.shop
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80

```

## 상용 클라우드 시스템에서 배포
### GOOGLE GKE
#### 클러스터 생성
GUI 환경에서 클러스터 생성
#### 생성하 클러스터로 CLI로 접근
```
gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project fluid-axe-315707
Fetching cluster endpoint and auth data.
kubeconfig entry generated for cluster-1.
```
#### 배포
minikube와 동일하게 배포 진행
```
kubectl get all --namespace app1
NAME                              READY   STATUS    RESTARTS   AGE
pod/deployment-5f5d7579f9-5w7pm   1/1     Running   0          32m
pod/deployment-5f5d7579f9-5x2bt   1/1     Running   0          32m
pod/deployment-5f5d7579f9-8cbrk   1/1     Running   0          32m
pod/deployment-5f5d7579f9-9t2tv   1/1     Running   0          32m
pod/deployment-5f5d7579f9-w5gf8   1/1     Running   0          32m

NAME                   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/app1-service   ClusterIP   10.8.4.250   <none>        80/TCP    32m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment   5/5     5            5           32m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-5f5d7579f9   5         5         5       33m
kubectl get ingress --namespace app1
NAME               CLASS    HOSTS            ADDRESS   PORTS     AGE
app1-tls-ingress   <none>   hojintest.shop             80, 443   25m
```
#### service까지 만들어지 상태, ingress를 통해 외부 노출및 확인
![image](https://user-images.githubusercontent.com/22079767/127019529-ee141ca6-5bbe-4a51-8b69-bce44fb98304.png)
![image](https://user-images.githubusercontent.com/22079767/127019685-1dd47f07-0418-4cb8-8ce8-17e1897bb2d8.png)

#### tls 설정 없이 tls-ingress 진행시 에러 발생
```
xxx@cloudshell:~/test (xxx)$ kubectl describe ingress --namespace app1
Name:             app1-tls-ingress
Namespace:        app1
Address:
Default backend:  default-http-backend:80 (10.4.0.3:8080)
TLS:
  secret-tls-ingress terminates hojintest.shop
Rules:
  Host            Path  Backends
  ----            ----  --------
  hojintest.shop
                  /   app1-service:80 (10.4.0.5:80,10.4.1.7:80,10.4.1.8:80 + 2 more...)
Annotations:      <none>
Events:
  Type     Reason  Age                From                     Message
  ----     ------  ----               ----                     -------
  Normal   Sync    91s (x2 over 91s)  loadbalancer-controller  Scheduled for sync
  Warning  Sync    3s (x12 over 34s)  loadbalancer-controller  Error syncing to GCP: error running load balancer syncing routine: error initializing translator env: secrets "secret-tls-ingress" not found
```
### temporaly delete tls testing
