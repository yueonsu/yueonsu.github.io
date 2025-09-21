---
title: "Istio 실습 (with GPT)"
date: 2025-09-21 13:00:00 +0900
categories: [Service Mesh,Istio]
tags: [Istio]
---

# 실습(GPT)
---

### ***minikube 설치***

---

```shell
$ brew install minikube
```

<br>

### ***Istioctl***

---

```shell
$ brew install istioctl
```

<br>

### ***minikube 실행***

---

첫번째 실행
```shell
$ minikube start --memory=8192 --cpus=4 --driver=docker
😄  Darwin 15.3.1 (arm64) 의 minikube v1.37.0
✨  유저 환경 설정 정보에 기반하여 docker 드라이버를 사용하는 중
- docker 데몬이 충분한 CPU/메모리 리소스에 액세스할 수 있는지 확인합니다.
- 문서: https://docs.docker.com/docker-for-mac/#resources

⛔  Exiting due to RSRC_INSUFFICIENT_CORES: Requested cpu count 4 is greater than the available cpus of 2
```
- docker resource 제한으로 실행 실패
- Rancher Desktop의 resource 제한을 좀 늘려서 재시도

<br>

두번째 실행
```shell
$ minikube start --driver=docker --cpus=4 --memory=8192
😄  Darwin 15.3.1 (arm64) 의 minikube v1.37.0
✨  유저 환경 설정 정보에 기반하여 docker 드라이버를 사용하는 중

💣  Exiting due to PROVIDER_DOCKER_NOT_RUNNING: "docker version --format <no value>-<no value>:<no value>" exit status 1: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
💡  권장: Start the Docker service
📘  문서: https://minikube.sigs.k8s.io/docs/drivers/docker/
```
- Rancher Desktop의 docker resource 설정 변경 후 실행
  - CPU: 4
  - Memory: 8GB
- docker daemon이 아직 뜨지 않아서 실행이 안됨.

<br>

세번째 실행
```shell
$ minikube start --driver=docker --cpus=4 --memory=8192
😄  Darwin 15.3.1 (arm64) 의 minikube v1.37.0
✨  유저 환경 설정 정보에 기반하여 docker 드라이버를 사용하는 중

❌  Exiting due to MK_USAGE: Docker Desktop has only 5921MB memory but you specified 8192MB
```
- 현재 Docker Desktop에 할당된 메모리가 **5.9GB**인데, Minikube 실행 옵션에서 **8GB**를 요청해서 실행이 안됨

<br>

네번째 실행
```shell
$ minikube start --driver=docker --cpus=2 --memory=4096
😄  Darwin 15.3.1 (arm64) 의 minikube v1.37.0
✨  유저 환경 설정 정보에 기반하여 docker 드라이버를 사용하는 중
📌  Docker Desktop 드라이버를 루트 권한으로 사용 중
👍  "minikube" 클러스터의 "minikube" primary control-plane 노드를 시작하는 중
🚜  기본 이미지 v0.0.48를 가져오는 중 ...
💾  쿠버네티스 v1.34.0 을 다운로드 중 ...
    > preloaded-images-k8s-v18-v1...:  332.38 MiB / 332.38 MiB  100.00% 8.90 Mi
    > gcr.io/k8s-minikube/kicbase...:  450.06 MiB / 450.06 MiB  100.00% 6.06 Mi
🔥  docker container (CPUs=2, 메모리=4096MB) 를 생성하는 중 ...
🐳  쿠버네티스 v1.34.0 을 Docker 28.4.0 런타임으로 설치하는 중
🔗  bridge CNI (Container Networking Interface) 를 구성하는 중 ...
🔎  Kubernetes 구성 요소를 확인...
    ▪ 이미지 gcr.io/k8s-minikube/storage-provisioner:v5 사용 중
🌟  애드온 활성화 : storage-provisioner, default-storageclass
🏄  끝났습니다! kubectl이 "minikube" 클러스터와 "default" 네임스페이스를 기본적으로 사용하도록 구성되었습니다
```
- 실행 설정 변경 후 성공
  - --cpus=2
  - --memory=4096
  
<br>

### ***Istio 설치***

---

```shell
$ istioctl install --set profile=demo -y
```
- demo 프로파일은 학습/테스트용으로 여러 기능(mTLS, ingress gateway 등)이 다 켜져 있음.

<br>

### ***namespace 레이블링(?)***

---

```shell
$ kubectl create namespace istio-test
$ kubectl label namespace istio-test istio-injection=enabled
```
- 이 네임스페이스에 배포하는 Pod는 자동으로 Envoy Proxy(sidecar)가 주입됨.

<br>

### ***sample application 배포***

---

```shell
# 배포
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -n istio-test
# 확인
$ kubectl get pods -n istio-test
```
- 각 Pod 옆에 istio-proxy 컨테이너(sidecar)가 붙어 있을 겁니다.

<br>

### ***Istio Ingress Gateway 접근***

---

```shell
# 1. 게이트웨이 리소스 생성
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml -n istio-test

# 2. 외부 접속용 주소 확인
$ kubectl get svc istio-ingressgateway -n istio-system

# 3. minikube 확인
$ minikube tunnel
```

<br>

### ***(?)***

---

- 네임스페이스 레이블링 (Kubernetes 자체 기능)
  - Kubernetes에서 네임스페이스 레이블링(namespace labeling) 은 네임스페이스 객체에 라벨(label) 을 붙여서 정책이나 기능을 적용하기 위한 과정.
  - 사용 시 이점:
    - 그룹핑 / 선택적 제어
    - 자동화와 정책 적용
    - 환경 구분
    - 운영 편의성 (해당 라벨의 파드만 삭제)
    - 확장성과 유연성

<br>

### ***요약***

---

GPT 통해서 실습하며 이해를 하려 했으나, 역시 직접해보지 않으니 확 와닿진 않음.  
조만간 직접 샘플 프로젝트로 실습해봐야겠다.
