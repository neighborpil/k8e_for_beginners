# k8e_for_beginners
hands on project to train K8E


## kubectl
- Kubernetis Controller
- 모든 쿠버네티스 기본 명령어
- 버전보기
```
$ kubectl version
```

### deploy
1. 컨테이너 생성
```
$ kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
```

2. Expose
```
$ kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
```
   


#### 도커 이미지는 어디에 있나?
- https://hub.docker.com/u/neighborpil 여기에 있다

#### 매번 새로운 클러스터를 생성하는 것이 귀찮다면
- 아래의 방식으로 클러스터의 노드를 0으로 감소시킨 후 다시 시작할때 노드의 숫자를 증가시키면 됨

```
-- make it zero
$ gcloud container clusters resize --zone <name_of_zone> <name_of_your_cluster> --num-nodes=0
- increase nodes
$ gcloud container clusters resize --zone <name_of_zone> <name_of_your_cluster> --num-nodes=3


-- make it zero
$ gcloud container clusters resize --zone us-central1 autopilot-cluster-1 --num-nodes=0
- increase nodes
$ gcloud container clusters resize --zone us-central1 autopilot-cluster-1 --num-nodes=1
```

### 이벤트 보기
- 클러스터에 서 일어난 이벤트를 볼 수 있다
```
$ kubectl get events
```

- 예시
```
8m7s        Normal    Scheduled                                pod/hello-world-rest-api-6755b7c4b7-5h7pb           Successfully assigned default/hello-world-rest-api-6755b7c4b7-5h7pb to gk3-autopilot-cluster-1-pool-2-228581d3-lf2p -- 컨테이너 배포 시작
7m57s       Normal    Pulling                                  pod/hello-world-rest-api-6755b7c4b7-5h7pb           Pulling image "in28min/hello-world-rest-api:0.0.1.RELEASE" -- 이미지 pulling
7m43s       Normal    Pulled                                   pod/hello-world-rest-api-6755b7c4b7-5h7pb           Successfully pulled image "in28min/hello-world-rest-api:0.0.1.RELEASE" in 2.049s (14.758s including waiting). Image size: 88504325 bytes.
7m41s       Normal    Created                                  pod/hello-world-rest-api-6755b7c4b7-5h7pb           Created container hello-world-rest-api
7m40s       Normal    Started                                  pod/hello-world-rest-api-6755b7c4b7-5h7pb           Started container hello-world-rest-api
9m11s       Normal    SuccessfulCreate                         replicaset/hello-world-rest-api-6755b7c4b7          Created pod: hello-world-rest-api-6755b7c4b7-5h7pb
9m13s       Normal    ScalingReplicaSet                        deployment/hello-world-rest-api                     Scaled up replica set hello-world-rest-api-6755b7c4b7 to 1
7m7s        Normal    ADD                                      service/hello-world-rest-api                        default/hello-world-rest-api
7m7s        Normal    EnsuringLoadBalancer                     service/hello-world-rest-api                        Ensuring load balancer
7m7s        Normal    UPDATE                                   service/hello-world-rest-api                        default/hello-world-rest-api
7m6s        Normal    DNSRecordProvisioningSucceeded           service/hello-world-rest-api                        DNS records updated
6m31s       Normal    EnsuredLoadBalancer                      service/hello-world-rest-api                        Ensured load balancer
```


### 쿠버네티스 조회(kubectl get)
- 현재의 파드 상황을 보여줌(pod or pods 양쪽 다 사용해도 된다)
```
$ kubectl get pods
```

- 현재의 레플리카셋의 상태를 보여줌
```
$ kubectl get replicaset
```

- 현재 서비스를 보여줌
```
$ kubectl get service
```

- 현재 배포 상태를 보여줌
```
$ kubectl get deployment
```

#### 쿠버네티스 컨셉
- Single responsibility principle
- One concept to one responsibility
- 


## Pod
- k8e에서 생성하고 관리 할 수 있는 가장 작은 배포 단위.
- 한 파드는 하나 이상의 컨테이너를 포함할 수 있음
- 이 컨테이너들의 묶음을 Pod라고 하며 같은 네트워크 네임스페이스와 스토리지를 공유
- 한글로는 파드라고 읽음
- Pod 없이는 배포할 수 없다
- Pot 정보 간단설명
    + 각각의 pod는 unique ip를 가진다
    + READY: 전체 컨테이너의 개수와 ready상태인지를 나타낸다
```
$ kubectl get pods -o wide


NAME                                    READY   STATUS    RESTARTS   AGE   IP             NODE                                           NOMINATED NODE   READINESS GATES
hello-world-rest-api-6755b7c4b7-5h7pb   1/1     Running   0          26m   10.21.128.11   gk3-autopilot-cluster-1-pool-2-228581d3-lf2p   <none>           <none>
```

- 쿠버네티스 pod의 상세설명 보기(describe 명령어)
```
$ kubectl get pods -- pod의 이름 표시

NAME                                    READY   STATUS    RESTARTS   AGE
hello-world-rest-api-6755b7c4b7-5h7pb   1/1     Running   0          31m

$ kubectl describe hello-world-rest-api-6755b7c4b7-5h7pb


Name:             hello-world-rest-api-6755b7c4b7-5h7pb
Namespace:        default -- isolation에 관련, dev, qa, production 분리할때
Priority:         0
Service Account:  default
Node:             gk3-autopilot-cluster-1-pool-2-228581d3-lf2p/10.128.0.6
Start Time:       Sat, 11 Jan 2025 08:24:56 +0000
Labels:           app=hello-world-rest-api
                  pod-template-hash=6755b7c4b7
Annotations:      <none> -- 배포시의 이름같은 것
Status:           Running -- 현재의 상태
SeccompProfile:   RuntimeDefault
IP:               10.21.128.11
IPs:
  IP:           10.21.128.11
Controlled By:  ReplicaSet/hello-world-rest-api-6755b7c4b7
Containers:
  hello-world-rest-api:
    Container ID:   containerd://d912641ce70fabfedb794f3299b7f6483302f949dd7a42095ed43d1bb6c44c39
    Image:          in28min/hello-world-rest-api:0.0.1.RELEASE
    Image ID:       docker.io/in28min/hello-world-rest-api@sha256:00469c343814aabe56ad1034427f546d43bafaaa11208a1eb0720993743f72be
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 11 Jan 2025 08:25:23 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      ephemeral-storage:  1Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qhkrw (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-qhkrw:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 kubernetes.io/arch=amd64:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                From                                   Message
  ----     ------            ----               ----                                   -------
  Normal   TriggeredScaleUp  31m                cluster-autoscaler                     pod triggered scale-up: [{https://www.googleapis.com/compute/v1/projects/the-name-443314-c0/zones/us-central1-a/instanceGroups/gk3-autopilot-cluster-1-pool-2-228581d3-grp 0->1 (max: 1000)}]
  Warning  FailedScheduling  30m (x7 over 31m)  gke.io/optimize-utilization-scheduler  no nodes available to schedule pods
  Warning  FailedScheduling  30m                gke.io/optimize-utilization-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {node.kubernetes.io/not-ready: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Normal   Scheduled         30m                gke.io/optimize-utilization-scheduler  Successfully assigned default/hello-world-rest-api-6755b7c4b7-5h7pb to gk3-autopilot-cluster-1-pool-2-228581d3-lf2p
  Normal   Pulling           30m                kubelet                                Pulling image "in28min/hello-world-rest-api:0.0.1.RELEASE"
  Normal   Pulled            29m                kubelet                                Successfully pulled image "in28min/hello-world-rest-api:0.0.1.RELEASE" in 2.049s (14.758s including waiting). Image size: 88504325 bytes.
  Normal   Created           29m                kubelet                                Created container hello-world-rest-api
  Normal   Started           29m                kubelet                                Started container hello-world-rest-api
```



#### pod가 무어인지에 대한 설명 보기
```
$ kubectl explain pods

```

