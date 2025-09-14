---
layout: article
title: "[Docker] 기본 개념"
date: 2024-09-14 14:59 +0900
category: Product_Serving
tags: [Product Serving]
aside:
  toc: true
sidebar:
  nav: Serving
---
## 가상화

- 개발 시에는 주로 운영 서버가 아닌 Local에서 개발을 한다. 이 때, 개발을 진행한 Local서버와 Production 서버의 OS가 다를 경우 라이브러리 등을 설치할 때 다르게 진행을 해야 하고, 또한 OS가 같더라도 Local과 Production의 환경 변수 차이로 인해 제대로 동작하지 않을 수 있다.
- 이러한 문제를 해결하기 위해 README 등에 환경 설정을 기록하고 실행하지만, 이는 사람이 직접 수행해야 하므로 에러가 발생하기 쉽고, 여러 서버에서 반복해야 하는 번거로움이 있다.
- 이를 위해 **가상화**를 진행한다. **가상화란 서버환경까지 모두 한번에 소프트웨어화하여 어디서든 동일한 환경이 되도록 하는 것**이다. 이를 통해 특정 소프트웨어 환경을 만들어 Local과 Production서버에 그대로 사용함으로써, 개발과 운영 서버의 환경을 일치 시킬 수 있다.
- 과거에는 이 가상화를 위해 VM(Virtual Machine)을 사용하였다. VM이란 실제 물리적 컴퓨터(Host Machine) 위에 OS를 포함한 가상화 소프트웨어를 두는 방식이다.
- 하지만 이 VM은 OS 위에 OS를 하나 더 두기에 리소스를 굉장히 많이 사용한다는 단점이 있다. 이런 단점을 극복하기 위해 Container를 사용한다. Container는 **VM과 달리 OS 위에 OS를 두지않고, Host의 OS를 공유하기에 VM보다 더 가볍고 빠른 경량화된  가상화 프로세스이다.**

<img width="771" height="667" alt="스크린샷 2025-09-14 145417" src="https://github.com/user-attachments/assets/0838033a-7053-45a4-a031-7a55ea61761c" />

<img width="649" height="560" alt="스크린샷 2025-09-14 150529" src="https://github.com/user-attachments/assets/039887a3-8739-4b94-be2e-7ae84aa835e2" />

---

## Docker

- Docker는 Container 기술을 쉽게 사용할 수 있도록 나온 도구로서, Container를 활용하여 개발자가 Application을 가볍고, 이식 가능하며, 일관된 방식으로 Application과 그 실행 환경을 **빌드**, **배포**, **실행**할 수 있도록 도와주도록 설계된 오픈 소스 플랫폼이다.
- **Docker의 장점**
    1. **Portability**: 개발자 환경, 테스트 서버, Cloud서버에서 동일한 환경으로 설정 가능하다.
    2. **Scalability**: Docker Container는 필요한 만큼 늘릴 수 있어, 트래픽이 몰릴 경우 Horizontal Scaling으로 트래픽을 분산시킬 수 있다. 
    3. **Speed**:  Container는 VM과 달리, OS 커널을 공유하기 때문에 속도가 더 빠르다.
    4. **Efficiency**: Container는 필요한 라이브러리와 애플리케이션만 포함하기 때문에 리소스(메모리, CPU)를 VM보다 적게 사용한다.
    5. **Consistency**
- **Docker의 주요 구성 요소**
    1. **Docker Engine**: Container들을 관리하는 엔진으로 Container를 생성, 실행 관리하는 프레임 워크이다.
    2. **Docker Image**: Scripting된 Docker File로부터 만들어진 바이너리 파일로 **Read-Only**이다.
    3. **Docker Container**: 일반화된 Docker Image를 기반으로 실제 실행 중인 **인스턴스**로 Docker Container간의 Communication이 가능하다.
    4. **Docker File**: **Docker Image를 정의**하는 Script파일이다.
    5. **Docker Compose**: **여러 Docker Container를 실행**할 수 있는 도구다.
    6. **Docker Compose**: Docker Image를 저장, 공유하는 **Cloud 기반의 저장소**이다.
- **Docker의 Workflow**
    1. 개발 환경을 정의한 Docker File을 작성
    2. Docker Image를 Build한다.
    
    ```bash
    docker build -t {이미지 이름}
    ```
    
    1. Docker Image로부터 Container를 실행한다.
    
    ```bash
    docker build -d -p {연결할 포트번호} {이미지 이름}
    # -d: Demon(백그라운드)로 돌아가게 하는 것
    # -p: 포트를 연결하는 것
    ```
    
    1. Docker Image를 Docker Hub나 다른 저장소에 저장한다.
    
    ```bash
    docker push {저장할 이미지}
    ```
    
    1. Production에서 Container를 적용한다.
