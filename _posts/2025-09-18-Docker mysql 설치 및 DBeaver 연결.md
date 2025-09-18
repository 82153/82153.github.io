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
1. **Docker hub에서 mysql 이미지 가져오기**
    - Docker hub에서 mysql 8.0버전의 이미지를 pull한다.
    
    ```bash
    docker pull mysql:8.0
    ```
    

![스크린샷 2025-09-18 021054.png](attachment:0982a7ad-ed25-497d-8348-2b34b0f36fd0:스크린샷_2025-09-18_021054.png)

1. **Docker Container 실행**
    - pull한 mysql이미지를 통해 Container를 실행시킨다.
    
    ```bash
    docker run --name {Container 이름} \
    	-e MYSQL_ROOT_PASSWORD={사용할 비번} \ # 환경변수 설정
    	-d \ # 백그라운드(demon)으로 실행
    	-p {local port}:{container 내부 port} \
    	{Image 이름}:{Image Tag}
    ```
    

![image.png](attachment:916241b6-779e-4e6e-a060-26ff5ed54197:image.png)

1. **DBeaver에 연동하기**
    - 데이터베이스 → 새 데이터베이스 연결을 클릭 한 뒤에 mysql을 선택한다.
        
        ![스크린샷 2025-09-18 161922.png](attachment:07ffe303-2c9e-48bc-a36b-fc8b193ebd2b:스크린샷_2025-09-18_161922.png)
        
    - 이후에 내가 Docker Container 실행 시에 작성한 **local의 host**로 맞춰주고, 비번을 작성해준다.
        
        ![스크린샷 2025-09-18 162601.png](attachment:b7e8dd77-5992-49f7-93ea-9824ad2f677b:스크린샷_2025-09-18_162601.png)
        
    - 그 다음에 Drive properties로 들어가서 **allowPublicKeyRetrieval**을 true로 변경해주고 완료를 누르면 dbeaver에 연동은 끝났다.
        
        ![스크린샷 2025-09-18 162430.png](attachment:447eef82-f10d-42db-b7f8-79485867ecf1:스크린샷_2025-09-18_162430.png)
        
2. **예시  DB 다운 받기**
    - mysql에서 연습용으로 제공하는 employees db를 다운 받을 것이다.
    - 우선 https://github.com/datacharmer/test_db이 링크에 있는 Repository를 다운받고 압축을 해제한다.
    - 그리고 다운받은 Repository를 Docker Container 내부로 옮겨준다.
        
        ![image.png](attachment:d9e48540-273e-441d-95e6-c4c66e2a6cd0:image.png)
        
    - 그리고 mysql Container로 들어간다.
    
    ```bash
    docker exec -it mysql-db mysql -u root -p
    ```
    
    - 마지막으로 employees.sql 파일을 실행해준다.
    
    ```bash
    SOURCE /employees.sql;
    ```
    
    ![image.png](attachment:6f9ef80b-4866-4754-bbcf-d27cc49caa19:image.png)
    

![image.png](attachment:2b771326-3f75-49c2-9d6d-555e4860fac0:image.png)

- 확인해보면 제대로 DB를 가져온 것을 확인할 수 있다.