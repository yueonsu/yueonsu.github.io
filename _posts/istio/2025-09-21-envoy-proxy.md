---
title: "Envoy Proxy란?"
date: 2025-09-21 13:00:00 +0900
categories: [Service Mesh,Network,Infrastructure]
tags: [Istio,Data Plane,데이터 플레인]
---

### Envoy Proxy란?

---

1. 개념
   - 클라우드 네이티브 L7 프록시
   - Service Mesh의 데이터 플레인(Data Plane)에서 핵심 역할 수행
   - API Gateway, Load Balancer, Sidecar Proxy 등 다양한 용도로 활용 가능


2. 주요 특징
   - L7 프록시 
     - HTTP/gRPC 같은 애플리케이션 계층에서 동작하며, 세밀한 트래픽 제어 가능
   
   - 사이드카 패턴 지원
     - 서비스 옆에 붙어 동작 → 애플리케이션 수정 없이 트래픽을 제어

   - Observability (가시성)
     - 요청/응답에 대한 상세 메트릭, 로깅, 트레이싱 제공

   - Service Discovery & Health Check
     - 동적으로 백엔드 서비스 찾기 및 상태 점검 가능

   - TLS/mTLS(?) 지원
     - 서비스 간 통신 암호화 및 인증

   - 고성능
     - C++로 작성되어 빠른 처리 속도와 낮은 레이턴시 제공
   

3. Envoy의 주요 용도
   1. Service Mesh의 데이터 플레인 (예: Istio, Kuma, Consul)
   2. API Gateway로서 외부 트래픽을 내부 서비스에 라우팅
   3. Load Balancer 역할 (Round-robin, Least request 등 알고리즘 제공)
   4. Observability/Tracing 수집기


4. Istio와의 관계
   - Istio의 데이터 플레인은 Envoy Proxy로 구성됨
   - Istio의 컨트롤 플레인(istiod)이 정책·구성을 전달 → Envoy가 실제 트래픽 처리

### 핵심 정리

---

- ##### ***Envoy는 클라우드 네이티브 시대의 범용 L7 프록시이자, Service Mesh의 필수 구성 요소***


### (?)

---

- mTLS
  - Mutual Transport Layer Security의 줄임말로,
    클라이언트와 서버가 서로를 인증하고, 암호화된 통신 채널을 맺는 방식입니다.
    일반적인 **TLS(HTTPS)**는 서버만 인증하는 반면,
    mTLS는 서버와 클라이언트 모두 인증서(cert)를 주고받아 신뢰를 검증합니다.
