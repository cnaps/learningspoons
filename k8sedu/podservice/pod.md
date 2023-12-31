## 도커 , 쿠버네티스 실습
### 도커 실습
- [도커사용법 기본](https://github.com/cnaps/learningspoons/blob/main/dockeredu/docker1.md)
- [켄테이너 외부 접속](https://github.com/cnaps/learningspoons/blob/main/dockeredu/docker2.md)
- [이미지 생성 ,저장소 push,도커파일](https://github.com/cnaps/learningspoons/blob/main/dockeredu/docker3.md)
### 쿠버네티스 실습
- [파드,디플로이먼트,서비스,레플리카](https://github.com/cnaps/learningspoons/blob/main/k8sedu/podservice/pod.md)
- [인그레스](https://github.com/cnaps/learningspoons/blob/main/k8sedu/ingress/ingress.md)

### 사전 준비사항
- Docker Desktop 설치 https://docs.docker.com/desktop/mac/install/ .


- 환경 - 쿠버네티스 설치 : Enable Kubernets 설정
- [kubectl 설치] (https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-macos/)

- kubectl 버전 및 연결확인해보기
```
~  kubectl version --short
Flag --short has been deprecated, and will be removed in the future. The --short output will become the default.
Client Version: v1.27.2
Kustomize Version: v5.0.1
Server Version: v1.27.2
~  kubectl cluster-info
Kubernetes control plane is running at https://kubernetes.docker.internal:6443
CoreDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
~  kubectl get nodes
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   15d   v1.27.2
```
- 여러개의 클러스터를 접근하는 경우 아래와 같이 연결정보를 확인 후 docker-desktop으로 접속을 변경하도록 하자.
- kubectl config 확인
``` 
  > kubectl config current-context
  docker-desktop
```
- 도커 데스크탑으로 config 변경
``` 
  > kubectl config use-context docker-desktop
  Switched to context "docker-desktop".
```
  
### Pod,Replicaset,Deployment, Service 실습 
- ngix-pod.yaml작성
```
echo 'kind : Pod
apiVersion : v1
metadata :
  name : my-nginx-pod
spec:
  containers:
  - name : my-nginx-container
    image: nginx:latest 
    ports:
      - containerPort: 80' > ngix-pod.yaml

```

- pod 배포
```
kubectl apply –f nginx-pod.yaml
Pod/my-nginx-pod created
```

- 배포된 pod 확인

``` 
> kubectl get pods
NAME.           READY        STATUS      RESTARTS   AGE
My-nginx-pod    1/1          Running      0         96s

```

- 레플리카 셋 작성 및 배포
- replicaset-ngix.yaml 생성
```
kind : ReplicaSet
apiVersion : apps/v1
metadata:
  name : replicaset-nginx
spec:
  replicas: 3 
  selector: 
    matchLabels:
      app: my-nginx-pods
  template:
    metadata: 
      name: my-nginx-pods
      labels:
        app: my-nginx-pods
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80        
    
```
- 레플리카 셋 배포 
```
kubectl apply -f replicaset-nginx.yaml
replicaset.apps/replicaset-nginx created
```
- 레플리카셋 확인
```
kubectl get replicaset
NAME                         DESIRED   CURRENT   READY   AGE
hello-nginxdemo-7cd85d4c8f   1         1         1       100m
replicaset-nginx             3         3         3       113s
```
- 파드 확인
```
kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
my-nginx-pod                       1/1     Running   0          11m
replicaset-nginx-4qw8k             1/1     Running   0          2m40s
replicaset-nginx-hgp6r             1/1     Running   0          2m40s
replicaset-nginx-rlsx9             1/1     Running   0          2m40s
```

- deployment 생성 및 배포
- deployment.yaml 생성
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

- deployment 생성
```
kubectl apply -f deployment.yaml
deployment.apps/webapp1 created
```

```
kubectl describe deployment webapp1

Name:                   webapp1
Namespace:              default
CreationTimestamp:      Sat, 15 Jul 2023 19:54:06 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=webapp1
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=webapp1
  Containers:
   webapp1:
    Image:        katacoda/docker-http-server:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp1-6486c5c977 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  82s   deployment-controller  Scaled up replica set webapp1-6486c5c977 to 3
```

- 서비스 생성 
```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1

```
- 서비스 배포 
```
ubectl apply -f servcie.yaml
service/webapp1-svc created
```

```
kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        119m
webapp1-svc      NodePort    10.109.115.185   <none>        80:30080/TCP   80s
```


- 로컬 ip를 확인하여 테스트 해본다.
- 맥에서 로컬 ip확인 방법
- ipconfig getifaddr en0 수행 이명령은 en0 네트워크 인터페이스 ip 주소를 반환 
- curl http:로컬ip:30080를 수행해보면 여러개의 pod가 로드밴런싱 되는 것을 확인 할 수 있다.
```
 scant 🌙   curl http://192.168.0.31:30080
<h1>This request was processed by host: webapp1-6486c5c977-4f5zw</h1>
 scant 🌙   curl http://192.168.0.31:30080
<h1>This request was processed by host: webapp1-6486c5c977-4f5zw</h1>
 scant 🌙   curl http://192.168.0.31:30080
<h1>This request was processed by host: webapp1-6486c5c977-4f5zw</h1>
 scant 🌙   curl http://192.168.0.31:30080
<h1>This request was processed by host: webapp1-6486c5c977-zhcm8</h1>
 scant 🌙   curl http://192.168.0.31:30080
<h1>This request was processed by host: webapp1-6486c5c977-7nl6x</h1>
 scant 🌙   curl http://192.168.0.31:30080
<h1>This request was processed by host: webapp1-6486c5c977-4f5zw</h1>
 scant 🌙   curl http://192.168.0.31:30080
<h1>This request was processed by host: webapp1-6486c5c977-7nl6x</h1>
```

- 스케일 아웃 수행

```
scant 🌙   kubectl get deployment/webapp1
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   3/3     3            3           27m
scant 🌙   kubectl scale --replicas=5 deployment/webapp1
deployment.apps/webapp1 scaled
scant 🌙   kubectl get deployment/webapp1
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   4/5     5            4           28m
scant 🌙   kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
webapp1-6486c5c977-4f5zw           1/1     Running   0          28m
webapp1-6486c5c977-4k5hz           1/1     Running   0          23s
webapp1-6486c5c977-7nl6x           1/1     Running   0          28m
webapp1-6486c5c977-fgs48           1/1     Running   0          23s
webapp1-6486c5c977-zhcm8           1/1     Running   0          28m
```
