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

- kubectl get pods : 팟 정보 출력
- kubectl run <컨테이너명> --image=<이미지명> : 이미지를 사용하여 컨테이너 생성
- kubectl get pods -o wide : 팟정보 상세 출력
- kubectl delete <팟이름>/<컨테이너명> : 팟 내부의 컨테이너 삭제
- kubectl describe pods : 팟들의 세부정보 출력

### 27 ~ 28.

- kubectl get replica : 레플리카 정보출력
- kubectl describe replica : 레플리카 상세정보 출력
- kubectl create -f <yaml파일> : 레플리카 생성
- kubectl delete replacaset <레플리카명> : 레플리카 제거
- kubectl edit replicaset/<레플리카명> : 현재 적용중인 레플리카 정보를 yaml파일 형식으로 만들어서 불러와서 연다. 수정시 수정한 부분 적용. 현재 돌고있는 팟에는 적용이 안되므로 팟은 제거해줄것.
- kubectl scale replicaset/new-replica-set --replicas=5 : 레플리카 개수 조정
