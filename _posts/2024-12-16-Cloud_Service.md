---
layout: article
title: "[Cloud]Cloud Service"
date: 2024-12-16 20:01 +0900
category: Product_Serving
tags: [Product Serving]
aside:
  toc: true
sidebar:
  nav: layouts
---
## Cloud Service

- Cloud Service란 서버, storage, 소프트웨어 등 필요한 IT 자원을 이용자가 준비할 필요 없이 제공 업체가 인터넷 연결을 기반으로 제공해주는 서비스이다.
- 웹, 앱 서비스를 만드는 경우, 자신의 컴퓨터로 서비스를 만들거나 IP로 누구나 접근 가능하게 할 수 있다. 하지만 이런 경우 내 컴퓨터가 종료되면 서비스 또한 종료된다.
- 이를 해결하기 위해 이전에는 서버실을 만들어서 운영을 하였지만, 서버의 확장성이나 물리적이 공간 부족, 갑작스러운 서버실 종료 등의 문제가 있었기에 Cloud 서비스가 발전했다.

## Cloud Service 공통 기술 개념

- **Computing Service(Server)**
    - 연산을 수행하는 서비스
    - 가상 컴퓨터 서버로 CPI나 Memory, GPU 등을 선택할 수 있다.
    - VM 인스턴스 생성 후, VM 인스턴스에 들어가서 사용 가능하다.
- **Serverless Computing**
    - Computing Service와 유사하지만, 서버 관리를 클라우드 쪽에서 진행해준다.
    - 코드를 제출하면, 그 코드를 가지고 서버를 실행해주는 형태이다.
    - 자동 확장 기능(Auto Scaling)도 제공된다.
- **Stateless Container**
    - Docker를 사용한 Container 기반으로 서버를 실행하는 구조이다.
    - Stateless는 이전에 배운 Docker의 Volume Mount처럼 컨테이너 삭제 시에, 내부의 저장된 데이터나 상태도 같이 삭제되는 것을 말한다.
    - 자동 확장 기능도 제공된다.
- **Object Storage**
    - 다양한 Object를 저장할 수 있는 저장소이다.
    - 다양한 형태의 데이터를 저장할 수 있으며, API를 사용해 데이터에 접근할 수 있다.
- **Database(DB)**
    - 서비스에서 사용하기 위한 데이터를 저장하는 공간이다.
    - 클라우드에서 제공하는 Database를 활용할 수 있다.
    - 저장된 데이터를 어떻게 활용하는 지에 따라 DB에 저장할 수도 있고, Object Storage에 저장할 수도 있다.
- **Data Warehouse**
    - 데이터 분석에 특화된 Database이다.
    - DB와 Object Storage에 있는 데이터 모두 Data Warehouse에 저장된다.
- **AI Platform**
    - AI Research, AI Develop 과정을 더 편리하게 해주는 제품이다.
    - MLOps 관련 서비스를 제공해준다.
- **Cloud 벤더 별 공통 기술 이름**

![스크린샷 2024-12-16 110559](https://github.com/user-attachments/assets/1e6a9599-9a2e-4850-9e66-982aaf9ebc59)

## Virtual Private Cloud(VPC)

- VPC는 실제로 같은 네트워크 안에 있지만 논리적으로 분리한 것이다. 즉, 여러 서버를 하나의 네트워크에 있도록 묶는 개념이라 볼 수 있다.
- 서브넷
    - VPC 안에서 여러 망으로 쪼갠 것이다.
    - Public과 Private로 구분된다.
- Routing
    - 경로를 설정하고 찾아가게 해준다.
- 예시
    
    ![스크린샷 2024-12-16 111404](https://github.com/user-attachments/assets/9f9c1117-ea97-4def-b014-45000f24cc12)
    
    - VPC를 하나의 아파트 단지라고 보자.
    - 그럼 경비원, 편의점, 아파트, 표지판 등이 있을건데, 편의점은 아파트 단지 안으로만 들어가면 누구나 들어갈 수 있다. 하지만 아파트 안으로는 아무나 들어갈 수 없다.
    - 즉 아파트는 Private Subnet이 되고, 편의점은 Public Subnet이 되는 것이다.
    - 또한 표지판은 어디에 무엇이 있는지 경로를 알려주는 Routing이 되는 것이다.
