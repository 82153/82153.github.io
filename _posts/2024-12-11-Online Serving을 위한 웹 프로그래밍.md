---
layout: post
title: "[FastAPI] Background For FastAPI"
date: 2024-12-12 11:12 +0900
category: Product_Serving
tags: [FastAPI]
aside:
  toc: true
---
## Online Serving

- 실시간으로 데이터를 처리하고 **즉각적인 결과 반환**이 중요하다.
- 주로 Cloud나 On-Premise 서버에서 모델 호스팅 후, 요청 들어오면 모델이 예측을 반환하는 구조
- **구현 방법**
    - 직접 웹 서버 개발
        - Flask, **FaskAPI** 등을 사용해 서버 구축
    - Cloud 서비스 활용
        - AWS의 SageMaker, GCP의 **Vertex AI** 등을 사용
        - MLOps의 다양한 부분을 이미 클라우드 회사에서 구축해 제공
        - 하지만 직접 구축 대비 운영 비용이 더 나갈 수 있고, 해당 서비스에 의존성이 생기고 내부 구현 방식을 정확히 파악 못할 수도 있다.
    - 오픈 소스 활용
        - Tensorflow Serving, Torch Serve, MLFlop, BentoML 등을 사용
        - 다양한 방식으로 개발 가능하지만, 매번 추상화 패턴을 가질 수 있음

## Server Architecture

- **모놀리스 아키텍처**
    - 하나의 큰 서버로 개발한다.
    - 일반 서비스와 ML 서비스 코드를 포함한 모든 로직이 하나의 거대한 코드 베이스에 저장된다.
    - 배포하는 것은 프로젝트 하나이다.
    - 장점
        - 서비스 개발 초기에는 단순하고 직관적
        - 관리할 코드 베이스가 하나이기에 심플하다.
    - 단점
        - 모든 서비스가 하나의 저장소에 저장된다.
        - 나중에 너무 커져서 이해하기 어려워 진다.
        - 서비스 코드 간 결합도가 높아서, 추후에 수정이나 추가 개발하기 어려울 수 있다.
        - 의존성 및 환경을 하나로 통일해야한다.
- **MicroService Architecture**
    - 여러 개의 작은 서버로 개발
    - 모든 로직이 각각의 개별 코드에 저장
    - 배포해야 할 코드 프로젝트가 여러 개
    - 구조
        - Client는 하나의 Server에게 요청
        - Server는 이 요청을 처리할 각각의 내부적인 Server로 요청
        - 내부적인 Server들이 이 요청을 처리하고 요청했던 서버로 반환
        - Server는 이 응답을 받아 필요에 맞게 변환 후에 Client에게 반환
    - 장점
        - 거대한 코드 베이스를 작게 나눌 수 있음
        - 필요에 따라 각 담당 서버 단위로 스케일 업/다운이 가능
        - 의존성 및 환경을 담당 서버 별로 다르게 둘 수 있음
    - 단점
        - 전체적인 구조와 통신이 복잡해짐

## API(Application Programming Interface)

- 소프트웨어 응용 프로그램들이 서로 상호 작용하기 위한 인터페이스를 총칭하는 말
- 즉 **특정 소프트웨어에서 다른 소프트웨어를 사용할 때의 인터페이스**
- 웹 API, 라이브러리, OS 시스템 콜 등 다양한 종류가 존재한다.
- **Web API**
    - Web에서 사용되는 API로서 주로 HTTP를 통해 웹 기술을 기반으로 하는 인터페이스이다.
    - REST, GraphQL, RPC 등의 종류가 있다.
    - HTTP는 정보를 주고 받을 때, 지켜야 하는 통신 프로토콜(약속)이다.

## Rest API(Representational State Transfer)

- 현대의 대부분의 서버들이 채택하는 API 방법으로 **자원을 표현하고 상태를 전송하는 것에 중점을 둔 API**
- 또한 각 요청이 어떤 동작이나 정보를 위한 것인지를 요청 모습을 보고 추론이 가능하다.
- 기본적인 데이터 처리로 추가(Create), 조회(Read), 수정(Update), 삭제(Delete)가 가능하다.
- Method : 서버에 요청을 보내기 위한 방식

| Method | description |
| --- | --- |
| GET | 리소스를 조회 |
| POST | 리소스를 생성 |
| PUT | 생성된 리소스 하나를 전체 업데이트 |
| PATCH | 생성된 리소스 하나를 부분 업데이트 |
| DELETE | 생성된 리소스 하나를 삭제 |

```python
# ex
GET http://localhost:8080/users?name=dongook&Desc=false
```

- http://
    - 사용하는 프로토콜이다.
    - URL내의 Schema이다.
- localhost
    - URL내 Host부분
    - IP나 Domain Name이 될 수 있다.
    - localhost는 127.0.0.1이라는 예측된 Domain name
- 8080
    - URL 내 Port부분
    - 하나의 host내에서 여러 Port를 가질 수 있다.
    - 각 서버는 하나의 Port를 사용하여 네트워크 통신을 한다.
- users
    - URL 내 Path 부분
    - API 엔드 포인트라고도 불린다.
- ?name=dongook&Desc=false
    - Query Parameter로 추가 정보를 제공하거나 데이터를 필터링한다.
    - Key, Value 쌍으로 이뤄지고, &로 연결하여 여러 개를 사용할 수 있다.
