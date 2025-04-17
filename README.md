# docker_kubernetes
도커와 쿠버네티스 설치

OS : Ubuntu-22.04 기준

진행 단계는 아래와 같다.
1. Docker 설치	컨테이너                                 
2. Kubernetes 컴포넌트 설치 (kubeadm, kubelet, kubectl)
3. 클러스터 초기화 (kubeadm init)	
4. 워커 노드 조인 (kubeadm join)	멀티노드 클러스터 구성 시	
5. 네트워크 플러그인 설치 (CNI)	파드 통신 가능하게	
6. 테스트 및 배포	클러스터 운영


1. 도커 설치
   1-1. 도커 스크립트를 우분투 혹은 리눅스 OS에 .sh파일로 저장
   -> nano install-docker.sh
   1-2. 실행 권한부여
   -> chmod +x install-docker.sh
   1-3. 실행
   -> ./install-docker.sh
   
2. 쿠버네티스 설치
   2-1. 쿠버네티스 스크립트를 OS에 .sh파일로 저장
   -> nano install-kubernetes.sh
   2-2. 실행 권한부여
   -> chmod +x install-kubernetes.sh
   2-3. 실행
   -> ./install-kubernetes.sh

3. 클러스터 초기화 (kubeadm init)
   3.1 CNI(Container Network Interface) 플러그인 중 Cilium 사용 
   -> sudo nano /etc/containerd/config.toml
      파일 내에서 disabled_plugins 항목에 cri가 없어야 하고,
     plugins."io.containerd.grpc.v1.cri" 블록이 존재해야 함
       [plugins."io.containerd.grpc.v1.cri"]
           sandbox_image = "registry.k8s.io/pause:3.9"
   3.1 containerd 재시작
   -> sudo systemctl restart containerd
   3.2  kubeadm init 
   -> sudo kubeadm init --pod-network-cidr=10.0.0.0/16
   -> mkdir -p $HOME/.kube
      sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
   -> sudo crictl ps -a | grep etcd
   3.3 정상작동 확인
   -> sudo crictl info
   [3.3.1] 3.3에서 오류 발생시
   -> crictl 설정 파일 수정`ㅌㅌ
      sudo nano /etc/crictl.yaml
      runtime-endpoint: unix:///run/containerd/containerd.sock 로 수정 후 저장
      sudo crictl info 실행
   
---------------------------------------------------------------
현재까지 완료된 단계
1. 도커/containerd 설치
2. Kubeadm으로 클러스터 초기화
3. crictl 설정완료
4. kubeadm init 성공
남은 단계는 클러스터 사용준비 및 네트워크 구성과 마스터 정상화가 남음
-----------------------------------------------------------------

4. kubectl 설정(마스터 노드에서 kubectl 쓸 수 있게 설정)
   mkdir -p $HOME/.kube
   sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
5. Cilium 설치 (Pod 네트워크 구성)
   -> curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz
      tar xzvf cilium-linux-amd64.tar.gz
      sudo mv cilium /usr/local/bin
      cilium install
7.  
