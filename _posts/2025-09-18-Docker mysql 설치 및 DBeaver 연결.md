---
layout: article
title: "[Docker] Docker mysql 설치 및 DBeaver 연결"
date: 2025-09-18 17:03 +0900
category: Product_Serving
tags: [Product Serving]
aside:
  toc: true
sidebar:
  nav: Serving
---
## Docker Hub에서 mysql 이미지 불러오기 및 Container 실행

    - Docker hub에서 mysql 8.0버전의 이미지를 pull한다.
    
    ```bash
    docker pull mysql:8.0
    ```
    

    <img width="800" height="200" alt="스크린샷 2025-09-18 021054" src="https://github.com/user-attachments/assets/8e3f87fc-5b42-4423-ade6-4eb3709973dc" />

    - pull한 mysql이미지를 통해 Container를 실행시킨다.
    
    ```bash
    docker run --name {Container 이름} \
    	-e MYSQL_ROOT_PASSWORD={사용할 비번} \ # 환경변수 설정
    	-d \ # 백그라운드(demon)으로 실행
    	-p {local port}:{container 내부 port} \
    	{Image 이름}:{Image Tag}
    ```
    

    <img width="800" height="200" alt="스크린샷 2025-09-18 161554" src="https://github.com/user-attachments/assets/7a3bc6e9-be65-4b56-be4b-ef0b00404450" />

---

## DBeaver 연동하기

    - 데이터베이스 → 새 데이터베이스 연결을 클릭 한 뒤에 mysql을 선택한다.
        
        <img width="600" height="400" alt="스크린샷 2025-09-18 161922" src="https://github.com/user-attachments/assets/319e12e7-2b32-4279-8de5-584cf4a383e5" />

    - 이후에 내가 Docker Container 실행 시에 작성한 **local의 host**로 맞춰주고, 비번을 작성해준다.
        
        <img width="600" height="400" alt="스크린샷 2025-09-18 162601" src="https://github.com/user-attachments/assets/8fa6e17f-c92a-48df-8120-6819c8075ae1" />

    - 그 다음에 Drive properties로 들어가서 **allowPublicKeyRetrieval**을 true로 변경해주고 완료를 누르면 dbeaver에 연동은 끝났다.
        
        <img width="600" height="400" alt="스크린샷 2025-09-18 162430" src="https://github.com/user-attachments/assets/1c7b4b36-361a-4555-becf-cde07846bbe7" />

---

## Employees DB 다운 받기

    - mysql에서 연습용으로 제공하는 employees db를 다운 받을 것이다.
    - 우선 https://github.com/datacharmer/test_db이 링크에 있는 Repository를 다운받고 압축을 해제한다.
    - 그리고 다운받은 Repository를 Docker Container 내부로 옮겨준다.
        
        <img width="600" height="400" alt="스크린샷 2025-09-18 164553" src="https://github.com/user-attachments/assets/ebd05cc2-a320-4e12-8c6e-6f3567ab9709" />

    - 그리고 mysql Container로 들어간다.
    
    ```bash
    docker exec -it mysql-db mysql -u root -p
    ```
    
    - 마지막으로 employees.sql 파일을 실행해준다.
    
    ```bash
    SOURCE /employees.sql;
    ```

    <div style="display:flex; gap:10px;">
      <img src="https://github.com/user-attachments/assets/45dde98f-df19-4bc2-87e8-830b8c8f6c1b" width="400" />
      <img src="https://github.com/user-attachments/assets/cbd7941a-1c19-4acc-8670-b0e83167befc" width="400" />
    </div>

- 확인해보면 제대로 DB를 가져온 것을 확인할 수 있다.
