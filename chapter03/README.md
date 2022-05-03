
# 3.1 kubectl

## 3.1.2 기본사용법

### kubectl은 다음 형식으로 명령을 작성

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

- command: 자원에 실행하려는 동작. create, get, delete 등을 사용할 수 있음.
- TYPE: 자원 타입. pod, service, ingress 등을 사용할 수 있음.
- NAME: 자원 이름.
- FLAG: 부가적으로 설정할 옵션을 입력

### example

```bash
$ kubectl run echoserver --image=nginx --port=80  # 간단한 에코 서버를 동작
pod/echoserver created

$ kubectl expose pod echoserver --type=NodePort  # 파드들에 접근할 때 필요한 서비스 생성
service/echoserver exposed
```

### 생성 상태 확인

```bash
$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
echoserver   1/1     Running   0          49m

$ kubectl get service
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
echoserver   NodePort    10.98.0.152   <none>        8080:31121/TCP   107s
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP          4d15h
```

### 로컬 컴퓨터로 포트포워딩

```bash
$ kubectl port-forward svc/echoserver 8080:80   
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

### 테스트한 자원을 모두 삭제

```bash
$ kubectl delete pod,service echoserver
pod "echoserver" deleted
service "echoserver" deleted
```

### 자원이 정상적으로 삭제 되었는지 확인

```bash
$ kubectl get pod,service       
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d15h
```

service/kubernetes 는 kube-apiserver 관련 파트들을 가르킴

## 3.1.6 자동 완성

[LINK](https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/)

### bash

```bash
source <(kubectl completion bash) # bash-completion 패키지를 먼저 설치한 후, bash의 자동 완성을 현재 셸에 설정한다
echo "source <(kubectl completion bash)" >> ~/.bashrc # 자동 완성을 bash 셸에 영구적으로 추가한다
```

### zsh

```bash
source <(kubectl completion zsh)  # 현재 셸에 zsh의 자동 완성 설정
echo "[[ $commands[kubectl] ]] && source <(kubectl completion zsh)" >> ~/.zshrc # 자동 완성을 zsh 셸에 영구적으로 추가한다.
```

# 3.2 deployment 이용해 container 실행하기

## 3.2.1 kubectl run으로 container 실행하기

kubectl run으로 pod를 실행시킬 때 기본 컨트롤러는 deployment

### deployment를 이용해 nginx container 실행

```bash
$ kubectl create deployment nginx-app --image nginx --port=80
deployment.apps/nginx-app created
```

### 실행결과 확인

```bash
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-6f7d8d4d55-m269j   1/1     Running   0          11s
```

### deployment 확인

```bash
$ kubectl get deployments.apps 
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   1/1     1            1           76s
```

### 실행 중인 pod 개수 수정

```bash
$ kubectl scale deploy nginx-app --replicas 2
deployment.apps/nginx-app scaled

$ kubectl get pods                           
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-6f7d8d4d55-jgg8m   1/1     Running   0          3s
nginx-app-6f7d8d4d55-m269j   1/1     Running   0          2m7s

$ kubectl get deployments.apps                               
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   2/2     2            2           3m2s
```

### deployment 삭제

```bash
$ kubectl delete deployments.apps nginx-app
deployment.apps "nginx-app" deleted
```

## 3.2.2 템플릿으로 container 실행하기

```bash
$ kubectl apply -f "./chapter03/deployment/nginx-app.yaml"
deployment.apps/nginx-app created

$ kubectl get pod
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-79db49bb8f-n647q   1/1     Running   0          7s
```

# 3.3 클러스터 외부에서 클러스터 안 앱에 접근하기

kubenetes 내부에서 실행한 container를 외부에서 접근하려면 kubenetes의 _service_ 를 사용해야 함.

```bash
$ kubectl expose deployment nginx-app --type=NodePort
service/nginx-app exposed

$ kubectl get service  
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        4d16h
nginx-app    NodePort    10.109.203.76   <none>        80:31463/TCP   18s

$ kubectl describe service nginx-app 
Name:                     nginx-app
Namespace:                default
Labels:                   app=nginx-app
Annotations:              <none>
Selector:                 app=nginx-app
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.109.203.76
IPs:                      10.109.203.76
LoadBalancer Ingress:     localhost
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31463/TCP
Endpoints:                10.1.0.45:80
Session Affinity:         None
External Traffic Policy:  Cluster
```

사용한 자원 정리하기

```bash
$ kubectl delete -f "./chapter03/deployment/nginx-app.yaml" 
deployment.apps "nginx-app" deleted
```
