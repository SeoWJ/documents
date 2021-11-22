# Kubernetes

[Kubernetes - Youtube 따배쿠 강의](https://www.notion.so/Kubernetes-Youtube-117a5ef79ecb4ce0a085756b58df4dc3)

---

# Udemy :: Certified Kubernetes Administrator (CKA)

---

## Section 2 : Core Concepts

### 10. Cluster Architecture

![Untitled](Kubernetes%20e7b02427f3274abcbbdb9bf62bb08fd4/Untitled.png)

- Master
    - Manage, Plan, Schedule, Monitor Nodes
- Worker Nodes
    - Host Application as Containers
- kube-api server
    - 쿠버네티스의 최상위 매니지먼트 컴포넌트. 클러스터들의 작업들을 오케스트레이팅한다.
- Container Runtime Engine
    - DNS 네트워킹 솔루션이 컨테이너 형식으로 배포될 수 있으며, 그렇기 때문에 컨테이너에서 이것을 실행시킬수 있어야 한다. 이것이 Container runtime engine. 가장 많이 사용되는것이 Docker이다.
- kubelet
    - 각각의 워커노드를 하나의 배로 가정하면 선장이 필요하다. 선장은 마스터에게 정보를 송/수신하며 연락을 주고받을수 있어야 한다. 이 선장의 역할을 하는것이 kubelet. 클러스터에서 agent방식으로 사용한다.
    - kube-api server로부터 명령어를 듣고 컨테이너를 실행하거나 파괴하는 역할을 수행한다.
    - kube-api server로 status report를 전송한다.
- kube-proxy service
    - 워커노드에서 다른워커노드로의 통신을 담당한다.
    - 웹서버와 디비서버가 다른 노드에서 작동중임을 가정한다면 다른노드와의 통신이 반드시 필요할 것. 이런 작업을 책임지는 서비스이다.

### 11. ETCD for Beginners

- ETCD
    - 신뢰할 수 있는 key-value 저장방식. simple, secure, fast.
    - 중복된 키는 사용불가
    - curl 명령어로 다운로드 후, tar로 압축을 풀고 ./etcd 명령어로 실행가능.
    - 2379포트를 디폴트로 사용
    - ./etcdctl set key1 value1
    - ./etcdctl get key1 → return value1
    - ./etcdctl : 사용가능한 모든 명령어 리스트 설명

### 12. ETCD in kubernetes

- kubectl에 사용되는 커맨드는 모두 ETCD 서버에서 꺼내온다.
- 클러스터에 적용한 모든 명령어 및 상태변화는 ETCD 서버에 저장된다.
- 클러스터 셋업 방식에 따라 ETCD는 다른 방식으로 배포된다
    - deployed from stratch
        - 바이너리 파일을 다운로드하여 본인이 마스터 노드에 직접 설치해야한다.
    - using kubeadm tool
        - kubeadm을 사용할 경우 ETCD서버를 POD으로 kube-system namespace에서 실행시킨다.
- HA(High Availity)
    - 고가용성. 시스템이 지정된 시간동안 장애없이 계속 작동할 수 있는 능력.
    - 이중화로 구축하여 한 서버가 작업수행에 실패할 경우 다른 서버가 이를 대체할 수 있도록 구성.
    - 로드밸런싱이 필요함.
- ETCD in HA Environment
    - 마스터 노드를 여러개 사용
    - ETCD 인스턴스를 ...? (3:00 부분. 이해 못함. 마스터 노드를 여러개 사용하는 대신 ETCD를 하나만 사용하는 것인지? ETCD를 마스터노드마다 설치하여 여러개 사용하는 것인지? 여러개 사용한다면 동기화는 어떻게? 하나만 사용한다면 이게 HA가 맞는가? ETCD가 뻗으면? → ETCD도 여러개 둬서 서로 동기화하는느낌인듯 함.)

### 14. Kube-API Server

- kubelet으로부터 명령을 받으면, 유저를 인증하고, 명령을 검증함.
- 이후 명령에 대한 정보를 ETCD로부터 받아 실행
- 실행된 명령어는 노드가 없이 생성된 POD에 배정되고, 스케줄러가 적당한 워커노드를 찾아 배정
- 이후 API 서버는 ETCD클러스터에 정보를 업데이트. 업데이트한 정보를 맞는 워커노드의 kubelet으로 전송
- kubelet은 워커노드에 POD를 생성하고 container runtime engine에 애플리케이션 이미지 배포를 지시.
- 끝나면, kubelet은 상태를 API서버로 전송하고 Api서버는 전송된 데이터를 ETCD클러스터에 업데이트.

### 15. Kube Controller Manager

- 컨트롤러는 다양한 컴포넌트들의 상태를 지속적으로 모니터링하는 프로세스.
- Node Controller
    - 노드의 상태를 모니터링하고 애플리케이션의 작동을 지속하기 위해서 필요한 작업들을 수행. kube-api server를 통해서 수행.
    - 5초에 한번씩 모니터링
    - 만일 노드의 heartbeat가 끊어졌다면 40초까지는 기다려줬다가 unreachable로 표시.
    - 노드가 unreachable로 변환된다면 5분간 복구할 시간을 제공.
    - 실패한다면 팟을 지우고 정상적인 팟에 재할당.
- Replicaion Controller
    - replica set들의 상태를 모니터링하고 충분한 숫자의 팟들이 사용가능한지를 보장하는 역할.
    - 팟이 죽는다면 새로운 팟을 생성
- 이외에도 많은 컨트롤러들이 존재.
- 이러한 컨트롤러들은 Kube Controller Manager에서 관리

### 16. Kube Scheduler

- 어떤 팟이 어떤 노드로 가야할지 배정하는 책임을 진다.
- 하나의 팟을 해당 노드에 실질적으로 적재하는것은 kubelet의 몫.
- 스케줄러가 팟을 배정하는 방식은 best-fit방식. 남은 cpu가 가장 많은 곳으로 보낸다.

### 17. kubelet

- 노드를 클러스터에 등록
- 팟의 상태를 계속 모니터링
- kube-api server에 지속적으로 리포트를 보냄

### 18. Kube Proxy

- 팟끼리의 인터널 네트워크.

### 19. Recap - PODs

- 팟이란 쿠버네티스의 가장 작은 오브젝트 단위.
- 팟이란 애플리케이션의 싱글 인스턴스
- 부하가 심해져 앱을 스케일링 해야 하는 경우, 동일한 팟에 앱을 늘리는 것이 아니라 새로운 팟을 만들고 그 위에서 앱을 실행하는 것.
- 다만, 하나의 앱을 실행시키는 데 있어 필요한 서포팅 앱이 필요한 경우, 동일한 팟에 올려서 같은 생명주기를 갖도록 만들 수 있다.

### 20. YAML in Kubernetes

- 쿠버네티스는 YAML파일을 오브젝트 생성의 인풋으로 집어넣는다.

### 21 ~ 26.

```
kubectl get pods : 팟 정보 출력
kubectl run <컨테이너명> --image=<이미지명> : 이미지를 사용하여 컨테이너 생성
kubectl get pods -o wide : 팟정보 상세 출력
kubectl delete <팟이름>/<컨테이너명> : 팟 내부의 컨테이너 삭제
kubectl describe pods : 팟들의 세부정보 출력
kubectl run <컨테이너명> --port=<포트 넘버> : 컨테이너 생성 후 포트 
```

### 27 ~ 28.

```
kubectl get replica : 레플리카 정보출력
kubectl describe replica : 레플리카 상세정보 출력
kubectl create -f <yaml파일> : 레플리카 생성
kubectl delete replacaset <레플리카명> : 레플리카 제거
kubectl edit replicaset/<레플리카명> : 현재 적용중인 레플리카 정보를 yaml파일 형식으로 만들어서 불러와서 연다. 수정시 수정한 부분 적용. 현재 돌고있는 팟에는 적용이 안되므로 팟은 제거해줄것.
kubectl scale replicaset/new-replica-set --replicas=5 : 레플리카 개수 조정
```

### 30 ~ 32. Deployment

- 애플리케이션을 업데이트 할 경우, 한꺼번에 업데이트 하는 것 보다 순차적으로 하나씩 바꾸어가는것이 좋음. 쿠버네티스는 이것을 지원.
- 문제가 생길 경우 롤백 가능

```
kubectl create –f <yaml파일> : deployment 생성.
kubectl get deployments
kubectl get all : 모든것을 다 출력.
kubectl create deployment <디플로이 이름> --image=<이미지 이름> --replicas=<레플리카 숫자>
```

### 34 ~ 36. namespace

- 동일 네임스페이스 내에서는 이름만으로 호출할 수 있다.
- 타 네임스페이스의 요소를 사용하고자 할 때에는 정해진 양식을 따라야 한다. 이름.네임스페이스.svc.cluster.local // cluster.local -> 도메인 네임. Svc -> 서비스의 서브 도메인

```
kubectl get pods : 디폴트 네임스페이스의 팟들을 불러옴
kubectl get pods --namespace=kube-system : kube-system 네임스페이스 안의 팟들을 불러옴
kubectl create namespace <네임스페이스 명> : 네임스페이스 생성
kubectl create -f <yaml파일> --namespace=<네임스페이스명> : 팟 등을 생성하는 명령어를 사용할 때 
kubectl config set-context $(cubectl config current-context) --namespace=dev : dev를 기본 네임스페이스로 변경
kubectl get pods --all-namespaces : 모든네임스페이스의 팟을 불러온다.
```
- yaml파일 안에 네임스페이스를 명시하고자 할 경우 metadata: 아랫단에 작성

### 37 ~ 40. Services

- 쿠버네티스에서의 서비스란 팟, 레플리카셋, 디플로이와 같은 하나의 오브젝트다.
- 노드 위에서 웹 어플리케이션의 포트 to 포트 리퀘스트를 연결해주는 역할을 한다.
- 서비스는 세가지 타입이 존재
	- NodePort : 인터널 팟을 노드의 포트에서 접속가능하도록 한다.
	- Cluster IP : 클러스터 내부에 가상IP를 생성하여 서로 다른 서비스간 통신이 가능하도록 한다.
	- Load Balancer

```
kubectl create –f <yaml파일명> : 서비스 생성
kubectl get services : 정보확인
```

#### NodePort
```
NodePort YAML 예시

apiVersion: v1
kind: Service
metadata:
	name: myapp-service
spec:
	type: NodePort
	ports:
		- targetPort: 80
		port: 80
		nodePort: 30008
	selector:
		app: myapp
		type: front-end
```
- 만일 하나의 노드에 selector항목이 동일한 여러 개의 팟이 존재한다면, 서비스는 자동으로 모두를 연결하여 사용한다.
- 부하분산은 랜덤 알고리즘으로 사용.

#### Cluster IP

- 웹 어플리케이션을 사용할 경우 WEB, WAS, DB간의 연결이 필요하다. 이런 경우 사용 가능.
- 내부 팟들의 경우 IP주소가 고정되어 있지 않다. 문제가 생겨 꺼지고 새로 생성되는 경우 주소가 변동.
- 동일한 종류의 팟끼리 그룹화하여 하나의 인터페이스로 만들고 통신할 경우 랜덤하게 배정.

```
apiVersion: v1
kind: Service
metadata:
	name: back-end
spec:
	type: ClusterIP
	ports:
		- targetPort: 80
		port: 80
	selector:
		app: myapp
		type: back-end
```
- `kubectl run httpd --image=httpd:alpine --port=80 --expose` 팟을 생성하면서 동시에 80번 포트를 개방하고 클러스터ip에 연결

#### Load Balancer

- 프론트단에서 로드밸런싱을 하기 위해 사용
- 유저들은 하나의 URL을 원하나 여러 개의 노드를 사용하면 주소가 다 다를수밖에 없음. 이를 해결하기 위해 사용
- 몇몇 로드밸런서를 지원하는 클라우드 플랫폼에서만 사용가능
- 지원하지 않는 VMWare등에서 사용하는 경우 NodePort와 동일하게 기능함.
```
apiVersion: v1
kind: Service
metadata:
	name: myapp-service
spec:
	type: LoadBalancer
	ports:
		- targetPort: 80
		port: 80
		nodePort: 30008
	selector:
		app: myapp
		type: front-end
```

### 42 ~ 45. Imperative vs Declarative

- imperative : 스텝 바이 스텝으로 하나하나 지정하고 커맨드 날려서 설정하는 방식
- declarative : IaC와 유사한 개념
- yaml파일 경우 create, update, replacement 명령어로 적용할 경우 imperative이다. 기존에 중복된 이름이 있으면 실패할 것이고, 리플레이스도 기존의 것이 없다면 실패하는 등의 문제가 발생하기 때문이다. 그러나 apply로 적용할 경우 declarative이다. 알아서 설정하고 적용하기 때문이다. 업데이트가 필요할 경우에도 apply로 적용한다면 기존에 오브젝트가 있는 것을 스스로 파악하여 업데이트를 진행한다.

### Tip
- --dry-run : 커맨드를 테스트하는 경우 사용. `--dry-run=client` 옵션을 줄 경우 실제로 리소스를 만들지 않고 리소스가 생성가능한지, 커맨드가 맞았는지를 출력해준다.
- -o yaml : yaml포맷을 스크린에 띄워준다.

- `kubectl run nginx --image=nginx` nginx팟 생성
- `kubectl run nginx –image=nginx --dry-run=client –o yaml` 팟을 정의하는 yaml 파일 생성
- `kubectl create deployment --image=nginx nginx` nginx deployment 생성
- `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml` deployment yaml파일 생성.
- `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml` nginx-deployment.yaml 파일을 만들고 저장.
- `kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml` or `kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml` 서비스 생성 및 6379포트 사용 설정
- `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=clinet -o yaml` or `kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml` NodePort 생성 및 nginx port 80을 포트 30080에 연결

---

## Section 3 : Scheduling

### 50 ~ 51. Manual Scheduling

- 모든 팟들은 spec: 하단에 nodeName이라는 필드를 가지고 있다. 미설정이 디폴트값이다. 쿠버네티스가 자동으로 추가하는 부분임.
- 스케줄러는 팟들을 살펴보며 nodeName이 설정되지 않은 팟을 스케줄링 후보군에 집어넣는다. 이후 팟에 적합한 노드를 스케줄링 알고리즘으로 정의한다.
- 만일 스케줄러가 없다면, 팟들은 영원히 Pending 상태로 존재할 것.


### 53 ~ 55. Labels & Selectors

- Label은 오브젝트들을 그룹화하기 위한 수단.
- 그룹내에 특징을 추가해 줄 수 있다.
- Selector는 그룹들 내에서 필터링을 해 주는 기능이다.
- label yaml파일 예시
```
apiVersion: v1
kind: Pod
metadata:
	name: simple-webapp
	labels:
		app: App1
		function: Front-end
spec:
	containers:
		- name: simple-webapp
		image: simple-webapp
		ports:
			- containerPort: 8080
```
- 라벨은 너의 필요에 따라 니 맘대로 몇개든 추가해 줄 수 있다.
- `kubectl get pods --selector app=App1` 셀렉터 사용법 예시
- ReplicaSet의 yaml 파일에서는 labels: 부분이 metadata 아래, spec: - template: - metadata: 아래 총 두군데 위치해야 한다.
	```
	apiVersion: apps/v1
	kind: ReplicaSet
	metadata:
		name: simple-webapp
		labels:
			app: App1
			function: Front-end
	spec:
		replicas: 3
		selector:
			matchLabels:
				app: App1
		template:
			metadata:
				labels:
					app: App1
					functions: Front-end
			spec:
				containers:
					name: simple-webapp
					image: simple-webapp
	```

### 56 ~ 58. Taints and Tolerations

- Taint는 노드가 특정한 팟만을 올릴수 있도록 설정하는 것이다.
- Toleration은 Taint가 설정된 노드에 맞는 설정을 갖는것이다. Toleration이 맞는 팟 만이 Taint가 설정된 노드 위에서 동작할 수 있다.
- `kubectl taint nodes <node이름> key=value:taint-effect` Taint 설정 명령어
- Taint에는 3가지 종류가 존재
    - NoSchedule : 팟들이 노드 위에 스케줄링 되지 않는다.
    - PreferNoSchedule : 팟들을 해당 노드 위에 스케줄링 하지 않도록 노력하는 것. 보장되지는 않는다.
    - NoExecute : 팟들이 노드 위로 스케줄링 되지 않으며, 기존에 존재하던 팟들이 Tolerate하지 않을 경우 evict된다.
- `kubectl taint nodes <node이름> <key명>=<value명>:NoSchedule`
- 팟에 Toleration을 부여하는 yaml 예시
    ```
    apiVersion: v1
    kind: Pod
    metadata:
        name: myapp-pod
    spec:
        containers:
        - name: nginx-container
        image: nginx
    tolerations:
        - key: <key명>
        operator: "Equal"
        value: <value명>
        effect: NoSchedule
    ```
- K8S는 마스터 노드에는 팟들을 스케줄링 하지 않는다
- `kubectl taint nodes <Node명> <taint명>-` Taint 제거 명령어
