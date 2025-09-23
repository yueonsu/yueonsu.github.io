---
layout: post
title: k8s / Istio 실습
date: 2025-09-23 23:35:52 +0900
categories: [ Kubernetes,Istio ]
tags: [ Kubernetes,k8s,Istio,practice ]
---

# ***k8s / Istio 실습***

---

사전에 python fastapi로 간략한 api를 생성함.  
fastapi를 k8s에 배포하는 과정 기록.

<br>

##### ***목차***
> - k8s (minikube) 배포
> - helm으로 배포
> - namespace 분리 (backend-was)
> - istio 주입


<br>

### ***k8s 배포***

---

python fastapi dockerfile
```shell
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port",  "8080"]
```

<br>

minikube 환경 세팅
```shell
eval $(minikube docker-env)
docker build -t fastapi-demo:0.1 .
```

<br>

deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: fastapi-demo
  template:
    metadata:
      labels:
         app: fastapi-demo
    spec:
      containers:
      - name: fastapi-demo
        image: fastapi-demo:0.1
        imagePullPolicy: Never
        ports:
          - containerPort: 8080
```

<br>

service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: fastapi-service
spec:
  type: NodePort
  selector:
    app: fastapi-demo
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30080
```

<br>

k8s배포
```shell
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
확인
```shell
kubectl get pods
kubectl get svc
```
minikube 실행
```shell
minikube service fastapi-service --url
```

<br>

### ***helm으로 배포 변경***

---

helm 기본 경로
```shell
manifest/                 # Helm 차트 루트 디렉토리
├── Chart.yaml            # 차트 메타데이터 (이름, 버전, apiVersion 등)
├── values.yaml           # 기본 설정 값 (이미지, 서비스, replica 등)
└── templates/            # 쿠버네티스 리소스 템플릿이 들어가는 폴더
    ├── deployment.yaml   # Deployment 템플릿
    └── service.yaml      # Service 템플릿
```

<br>

Chart.yaml
```yaml
apiVersion: v2
name: fastapi-chart
description: A Helm chart for FastAPI demo
type: application
version: 0.1.0
appVersion: "0.1"
```
<br>

deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
  namespace: backend-was
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
```

<br>

service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
  namespace: backend-was
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Chart.Name }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.port }}
      nodePort: {{ .Values.service.nodePort }}
```

helm으로 배포
```shell
helm install fastapi manifest/
```

<br>

helm으로 pod 중지
```shell
helm uninstall fastapi
```

<br>

### ***Istio 주입***

---

namespace생성 및 istio 주입
```shell
kubectl create namespace fastapi-ns
kubectl label namespace fastapi-ns istio-injection=enabled
```
