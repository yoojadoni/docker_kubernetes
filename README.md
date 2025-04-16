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
