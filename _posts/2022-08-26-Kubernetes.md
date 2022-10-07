---
layout: single
title:  "Kubernetes 서버구성하기"
---

# Kubernetes

Hyperviser = VMWare, Hiper-V

- 쿠버네티스(K8s)  
그리스어로 조타수  
컨테이너화된 애플리케이션을 자동으로 배포  
스케일링 및 관리해주는 오픈소스 시스템  
구글 제작
멀티호스트 도커 플랫폼  
도커가 돌아가는 서버를 여러대로  
서버가 많아지다보니 많은 컨테이너를 컨트롤하기 어려움  
컨테이너 오케스트레이션 -  
서버1 - control plane(지휘자)  
서버2 - worker node1  
서버3 - worker node2  

계층|명칭|플랫폼
:--|:--:|:--
Layer6|Development Workflow Opinionated Containers|OpenShift, **Docker Cloud**, Cloud Foundary, Deis
Layer5|Orchestration/Scheduling Service Model|**Kubernetes**, Docker Swarm, Nomad, Diago
Layer4|Container Engine|**Docker**, Rocket, RunC, Osv
Layer3|Operating System|Ubuntu, RHEL, CoreOS
Layer2|Virtual Infrastructure|vSphere, EC2, GCP, Azure
Layer1|Physical Infrastructure|Raw Computer, Network, Storage

- 특징  
선언적 API - 서버 3개 실행해줘 하면 명령  
모니터링 하고 서버상태를 봐가며 웹서버 3개실행시킴


## 설치없이 실습하기

[쿠버네티스 플레이그라운드 접속](https://labs.play-with-k8s.com)  
깃 혹은 도커계정으로 가입한 후 4시간동안 쓸수 있음  

실행되고 나면 먼저 인스턴스 생성

1. Initializes cluster master node:

 kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16

 2. Initialize cluster networking:

kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml

 3. (Optional) Create an nginx deployment:

 kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml

 1 번을 수행하면 아래와 같이 차일드서버가 조인하는 명령정보나 출력된다.

 kubeadm join 192.168.0.8:6443 --token 4cwzyy.fxhhufzbsmczxnch \
    --discovery-token-ca-cert-hash sha256:2f09bc138785e6abe268a0f2c9ecacb12575761b6ea09f785500adb5114155f0  

이후 2번 명령어를 수행한다.  

2번도 완료되면 새로운 인스턴스를 생성하여 1번에서나온 조인 명령어를 수행한다.  
차일드 서버를 늘리려면 인스턴스를 추가로 생성하여 똑같이 명령어를 수행한다.

다시 마스터서버로 넘어와 "kubectl get nodes -o wide" 명렁어를 실행하면 연결된서버정보가 나온다.

## 쿠버네티스 직접설치하여 사용하기

CNI(Container Network Interface)
컨테이너간에 통신지원하는 인터페이스 받드시설치해야 통신이가능함  
다양한 솔루션이있음 플라넬, 칼리코, **위브넷**, 큐브라우터  

## 쿠버네티스 클러스터 구성
- control plane(master node)
  - 워커 노드들의 상태를 관리제어
  - single master
  - multi master(3,5개등)
- worker node
  - 도커 플랫폼을 통해 컨테이너를 동작하며 실제서비스 제공

- 가상머신 설치
  - 마스터 : 우분투 core=2,ram=2g,storage=32g, kube_master/기본***
    - 마스터에만 ui설치
    - $ sudo apt update
    - $ sudo apt install tasksel
    - $ sudo tasksel install ubuntu-desktop
    - $ sudo apt-get install --no-install-recommends ubuntu-desktop
    - $ sudo reboot
  - 워커 : 우분투 core=1,ram=1g,storage=32g, kube_worker/기본***
  - $ sudo apt-get install indicator-appmenu-tools (hud service not connected 오류 해결)

64.6 : master
64.4 : worker
64.7 : worker_sub

- Docker install
  - https://docs.docker.com/engine/install/ubuntu/
    - sudo apt-get update
    - sudo apt-get install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
    - sudo mkdir -p /etc/apt/keyrings
    - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    - echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    - sudo apt-get update
    - sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    - apt-cache madison docker-ce    "여기서 버전확인후 아래입력"
    - sudo apt-get install docker-ce=5:20.10.17~3-0~ubuntu-jammy docker-ce-cli=5:20.10.17~3-0~ubuntu-jammy containerd.io docker-compose-plugin
    - sudo docker run hello-world
    - systemctl enable docker
    - systemctl start docker
    - docker version

https://github.com/237summit/k8s_core_labs

- Kubernetes install
  - 환경설정
    - swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab
    - cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
    - sysctl --system
    - systemctl stop firewalld
    - systemctl disable firewalld
  - kubeadm(커맨드), kubectl(데몬), kubelet(명령어유틸) 설치
    - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
    - apt-get update
    - apt-get install -y apt-transport-https ca-certificates curl
    - curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
    - echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    - apt-get update
    - apt-get install -y kubelet kubeadm kubectl
    - apt-mark hold kubelet kubeadm kubectl
    - systemctl start kubelet
    - systemctl enable kubelet
  - control-plane 구성
    - <마스터> kubeadm init
      - 오류발생시
        - rm /etc/containerd/config.toml
        - systemctl restart containerd
        - kubeadm reset
        - kubeadm init
      - 오류발생시(
        - echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
    - 성공할경우 아래
    - mkdir -p $HOME/.kube
    - cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    - chown $(id -u):$(id -g) $HOME/.kube/config
    - 위에서 생성된 join 문 복사
    - kubectl get nodes
    - kubectl get pods --all-namespaces
  - worker node 구성
    - <워커> kubeadm reset
    - 마스터에서 생성된 join 문 붙여넣기
  - 설치확인

- Linux root Setting
  - sudo passwd root -> root 암호설정
  - su - -> root 로그인
  - 인증 후 su 권한으로 사용
- Ubuntu hostname 변경
  - hostnamectl set-hostname 변경할 이름
  - 재부팅 없이 hostname 실행하여 변경된 이름 확인

https://www.youtube.com/watch?v=r7TWWRL59aY

https://kubetm.github.io/k8s/02-beginner/cluster-install-case6/


마지막 해결책


파일수정
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf

Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/e`tc/kubernetes/kubelet.conf --cgroup-driver=cgroupfs"

kubeadm join 192.168.64.8:6443 --token vvfs74.yysnoa4dvk09myiy \
        --discovery-token-ca-cert-hash sha256:a8a0708e4d67ca7b66839833ee50fa759158126440e941bb15a8a0f18f8816c5

--cgroup-driver=cgroupfs

Kubeadm reset -> init

user@k8s-master:~/$ sudo rm /etc/containerd/config.toml
user@k8s-master:~/$ sudo systemctl restart containerd
user@k8s-master:~/$ sudo kubeadm init

임시 삭제해야함

CI/CD 환경 구축

- 독커

- 쿠버네티스
 = 설치중 어드민에 이슈생김

nSwitch 스크립트 심기

https://partner.nswitch.co.kr/advertiser/documents/pre_reservation/kr?nsw_camp_id=5740&nsw_track_id=260

    if("{$product_no}"==177)
  		new NSW_init_pr_param();


    <!-- nSwitch Tracker start -->
   <script type="text/javascript" src="https://scr.nsmartad.com/nswitch/npr_track/npr_track.js"></script>
   <script type="text/javascript">   
       new NSW_pre_reservation({track_id : '260', conv_id : '{$order_id}'});
       console.log("nswitch 전환로그 “ + '{$order_id}' );
    </script>
   <!-- nSwitch Tracker end -->

[https://bcho.tistory.com/1257](조대협 가이드)

- Pod, Volume, Service, namespace, Label, Controller
  - Pod : 하나 이상의 컨테이터를 포함하는 배포단위
  - Volume : 디스크, 컨테이너간에 공유되는 스토리지볼륨, 로컬 디스크 뿐만아니라 외장 네트워크(클라우드)도 가능
    - gitRepo : 볼륨 생성시 지정된 git 레파지토리의 특정 리비전의 내용을 클론을 이용해 내려받고 디스크볼륨 생성하는 방식
  - Service : 동일한 라벨로 만들어진 컨테이너들을 묶어서 로드벨런싱
  - Namespace : 클러스터 내의 서비스들을 목적별로 (논리적)분리, 복잡할경우 클러스터 분리를 추천
  - Label : 셀렉터로 리소스를 선택하여 집합의 개념으로 사용된다. (in, notin)
  - Controller : 기본 오브젝트를 생성하고 관리하는 역할
    - Replication Controller : Pod을 가동하고 관리하는 역할
      - Selector : Label을 기반으로 RC가 관리하는 Pod을 가져오는데 사용
      - Replica 수 : RC에 의해서 관리되는 Pod의 수, 모자르면 Pod 추가로 띄오고 남으면 삭제한다.
      - Pod Template : 추가로 가동하는 Pod을 만드는 생성정보
    - Replica Set : RC의 새버전
    - Deployment : RC와 Replica Set의 상위 추상화 개념 RC/RS 보다 이것이 좋다
      - 아래 배포작업을 RC/RS를 이용해 수동으로 하는게 아니라 자동화하는데 Deployment
    - 고급 컨트롤러
      - Daemon Set : 특정노드(장비SSD/GPU)를 선택적으로 배포
    - Job : 항상 실행된 상태가아니라 배치와 같이 한번 실행되고 끝나는 작업
    - Cron Jobs : 주기적으로 실행되는 작업
    - Stateful Set : 데이터베이스와 같이 상태를 가진 Pod을 지원하기 위한 작업

- 컨테이너 배포방법
  - 블루/그린 배포 : 기존서비스와 동일한 Pod들을 새로운 템플릿으로 새로 만들고 한번에 교체
  - 롤링 업그레이드 : Pod을 하나씩 업그레이드

- 마스터 : 클러스터 전체를 컨트럴하는 시스템, 크게 API서버, 스캐쥴러, 컨트롤러 매니져, etcd로 구성됨
  - API 서버 : 쿠버네티스 모든 기능을 RestAPI로 제공하고 명령을 처리한다.
  - Etcd : 쿠버네티스 클러스터의 데이터베이스 역할을 하는 서버로 설정값이 저장되어있음
  - 스케쥴러 : Pod, 서비스등 각 리소스를 적당한 노드에 할당하는 역할
  - 컨트롤러 매니져 : 컨트롤러를 생성하고 각 노드에 배포하고 관리하는 역할
  - DNS : 리소스의 엔드포인트를 DNS로 맵핑관리, Pod이나 서비스등은 IP를 배정받는데 동적으로 생성되는 리소스라 항상 병경되기 때문에 내부에 DNS를 넣는방식으로 해결함
- 노드 : 마스터에 의해 명령받고 실제 워크로드를 생성하여 서비스함, Kubelet, Kube-proxy, cAdvisor 그리고 컨테이너 런타임에 배포단위
  - Kubelet : 배포되는 에이전트로, 마스터의 API서버와 통신을 하면서, 수행해야 할 명령을 받아서 수행, 볼륨 마운트, 컨테이너 라이프사이클 검사, 상태를 마스터에 전달하는 역할
  - Kube-proxy : 노드로 들어오는 트래픽을 적당한 컨테이너로 라우팅, 로드밸런식 기능, 패킷포워드, 마스터와의 네트워크 통신을 관리
  - Container runtime : 컨테이너를 실행하는 런타임, 보통 도커를 생각하는데 이외 rkt, Hyper등 다양하게 있다.
  - cAdvisor : 각 노드에 기동되는 모니터링 에이전트, 상태 성능등의 정보를 마스터에 전달.

- kubectl
  - kubectl [command] [TYPE] [NAME] [flag]
    - command : object에 실행할 명령(create, get, delete, edit..)
    - TYPE : node, pod, service
    - NAME : 자원의 이름
    - flag : 부가적 설정 --help -o Optional
  - BASH
    - source <(kubectl completion bash)
    - source <(kubeadm completion bash)
  - 명령어
    - kubectl --help
    - kubectl command --help
    - kubectl run <자원이름> <옵션>
    - kubectl create -f obj.yaml
    - kubectl apply -f obj.yaml
    - kubectl get <자원이름> <객체이름>
    - kubectl edit <자원이름> <객체이름>
    - kubectl descript <자원이름> <객체이름>
    - kubectl delete pod main
  - 실습하기
    - kubectl get nodes : 노드정보보여주기
    - kubectl get nodes -o : 노드상세정보보여주기
    - kubectl describe node master.example.com : 노드상태를 자세하게 보여줌
    - kubectl run webserver --image=nginx:1.14 --port 80 : 웹서버 생성
    - kubectl get pods -o wide : pod 상태보기
    - kubectl describe pods

대시 한개(-) : 시스템5계열
대시 두개(--) : bsd계열

```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt-get install -y docker.io
```

## Create required directories

sudo mkdir -p /etc/systemd/system/docker.service.d

## Create daemon json config file

sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

## Start and enable Services

sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker

## 디플로이먼트 매니패스트 구조

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
```

- apiVersion : 쿠버네티스 리소스버전, 서로 다른 버전의 리소스와 작동가능
- kind : 리소스 또는 API 오브젝트를 지정
  - Deployment : 마이크로서비스 배포
  - Service : 서비스를 외부로 노출
  - ServiceAccount : 마이크로서비스에 대한 ID, 액세스권한을 가짐
  - Secret : 시크릿관리기능
  - Pod : 
  - Role : 규칙으로 정의된 리소스 권한집합
  - RoleBinding : 사용자, 사용자 그룹, 서비스 어카운트를 롤과 연결
  - HorizontalPodAutoscaler : 마이크로서비스 확장시 이용
- metadata : 키 값 문자열 쌍인 리소스이름, 레이블 집합, 특정자원참조, 라벨링
- spec : 레플리카셋 스펙, 파드 템플릿 존재, 레이블이 일치하는 파드를 관리
- template
  - metadata : 레이블을 지정하는 곳
  - spec : 파드의 컨테이너를 설명

