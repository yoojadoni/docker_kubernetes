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
3. helm 설치
   -> sudo apt-get update
      sudo apt-get install -y apt-transport-https curl gnupg
      curl -fsSL https://baltocdn.com/helm/signing.asc | sudo gpg --dearmor -o /usr/share/keyrings/helm.gpg
      echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | \
      sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
      sudo apt-get update
      sudo apt-get install -y helm
      helm version
4. Cilium 설치
   -> pod간의 통신을 위한 네트워크 솔루션
      sudo swapoff -a
      sudo rm -rf /etc/cni/net.d
      rm -f $HOME/.kube/config
      sudo systemctl restart containerd
      sudo mkdir -p /etc/containerd
      containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
      sudo nano /etc/containerd/config.toml
      -> [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true 로 변경
      sudo systemctl restart containerd
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      4.1. helm 저장소에 Cilium 추가
        helm repo add cilium https://helm.cilium.io/
        helm repo update
      4.2. Cilium 설치
        helm install cilium cilium/cilium --version 1.14.0 \
        --namespace kube-system \
        --set kubeProxyReplacement=true \  
        --set k8sServiceHost=192.168.100.10(마스터노드 IP) \
        --set k8sServicePort=6443
       설명: kubeProxyReplacement=true 를 주는 이유: Cilium이 kube-proxy 없이 kube-proxy 역할도 해주기 때문
      4.3. 설치 확인
      kubectl get pods -n kube-system
      모든 Cilium 관련 Pod가 Running 상태인지 확인하고, coredns도 Running으로 바뀌어야 함.
      만약 :
         Name                                      READY   STATUS    RESTARTES  AGE
         coredns-76f75df574-bd7pr                  1/1     Running   0          35m
         coredns-76f75df574-c2szz                  1/1     Running   0          35m
         etcd-culturegift-web                      1/1     Running   0          35m
         kube-apiserver-culturegift-web            1/1     Running   0          35m
         kube-controller-manager-culturegift-web   1/1     Running   0          35m
         kube-proxy-kssl9                          1/1     Running   0          35m
         kube-scheduler-culturegift-web            1/1     Running   0          35m
      위와 같은것외에 것이 Running이 아니고 Pendding 이면
      Cilium Operator 복제본 수 줄이기 (1개만 유지)  replica 수를 1개로 줄이기
      kubectl -n kube-system get deployment cilium-operator
      에서 replicas를 1로 줄이기
      kubectl -n kube-system scale deployment cilium-operator --replicas=1 
      kubectl delete pod <Name> -n kube-system
5. 프라이빗 Docker Registry 생성
-> docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  registry:2

윈도우 호스트에서 포트 포워딩 설정
Hyper-V에서 우분투 VM이 윈도우 호스트 포트 5000과 연결되도록 포트 포워딩을 설정하면,
Jenkins에서 10.5.5.12:5000 으로 접근 가능하게 만들 수 있어야함
netsh interface portproxy add v4tov4 listenport=5000 listenaddress=10.5.5.12 connectport=5000 connectaddress=192.168.100.10
netsh interface portproxy add v4tov4 listenport=5000 listenaddress=10.5.5.12 connectport=5000 connectaddress=192.168.100.0

참고 : 추가: 프라이빗 Registry가 HTTP라면
Jenkins 쪽 Docker 데몬에 아래 설정도 필요
{
  "insecure-registries": ["192.168.100.10:5000"]
}

6.  마스터노드에 워커노드 연결 및 워커노드 생성
   참고 : 기존 클러스터 초기화 (재설정)
         -> sudo kubeadm reset -f
            sudo rm -rf /etc/kubernetes/kubelet.conf
            sudo rm -rf /etc/kubernetes/pki/ca.crt
            sudo rm -rf ~/.kube
            sudo rm -rf /etc/cni/net.d
            sudo systemctl restart containerd
           Kubernetes 클러스터 재초기화
         -> sudo kubeadm init --apiserver-advertise-address=192.168.100.10 --pod-network-cidr=10.0.0.0/16
   6-1. 마스터 노드에서 Join Token 생성
      -> sudo kubeadm token create --print-join-command
   6.2. 6-1에서 발급된 명령어 실행
      -> kubeadm join 192.168.100.10:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      와 비슷한 명령줄 생성됨.
    
8.  


