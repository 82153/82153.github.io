---
layout: article
title: "[Docker] 기본 Command"
date: 2025-09-14 15:53 +0900
category: Product_Serving
tags: [Product Serving]
aside:
  toc: true
sidebar:
  nav: Serving
---
## General Command

1. **Docker 버전 확인**
    
    ```bash
    docker --version
    ```
    
2. **Docker의 정보 확인(Container, Image의 개수, 저장소 세부사항 등)**
    
    ```bash
    docker info
    ```
    
3. **Docker의 Command와 옵션 확인**
    
    ```bash
    docker help
    ```
    

---

## Docker Image 관련 Command

1. **Local Machine에 있는 모든 Image 리스트 확인**
    
    ```bash
    docker image
    ```
    
2. **Docker Hub이나 다른 저장소에서 이미지 불러오기**
    
    ```bash
    docker pull {image 이름}
    ```
    
3. **Docker Image 만들기**
    - .은 현재 directory의 docker file을 기반으로 만들라는 것.
    - -t는 태깅하는 것으로 이름을 붙이는 것이다.

    ```bash
    docker build -t {image 이름} .
    ```
    
4. **Docker의 다른 이름이나 버전을 태깅하는 법**
    - 하나의 이미지에는 여러 개의 tag가 붙을 수 있다.
    - 즉, 아래 코드는 새로운 tag를 추가하는 것이지 기존 tag를 없애고 새로 tag를 지정하는 것이 아니다.
    - source image: tag를 붙일 이미지
    - target image:tag : 새로 붙일 image이름과 tag
    
    ```bash
    docker tag {source image} {target image:tag}
    ```
    
5. **Docker Image를 Docker Hub이나 저장소에 push**
    - 저장소가 연결되어 있으면 해당 저장소로 저장될 것이고, 아니면 Docker Hub에 저장됨
    
    ```bash
    docker push {image 이름}
    ```
    
6. **Docker Image 삭제**
    - list로 넣어 한번에 여러 개 삭제 가능
    - docker rmi image1 image2 image3 이런 식으로 한번에 삭제 가능    
    
    ```bash
    docker rmi {Image id / Image 이름}
    ```
    

---

## Docker Container 관련 Command

1. **현재 작동중인 Container 확인**
    - 모든 container를 보고 싶을 때는 -a 옵션 사용
    - docker pd -a
    
    ```bash
    docker ps
    ```
    
2. **이미지로부터 Docker Container 생성**
    - 만약 Local에 없는 Image를 run한다면 docker hub에서 자동으로 다운 받는다.
    - -d: 백그라운드(Demon)에서 실행
    - -p {host-port}:{container-port}: 호스트 포트와 컨테이너 포트 연결
    - --name {container 이름}: 이름 지정
    - -v {host-port}:{container-port}: volume(저장소) 연결
    
    ```bash
    docker run {option} {Image 이름}
    ```
    
3. **Container 삭제**
    - -f옵션으로 현재 실행 중인 container도 삭제 가능(rm -f)
    
    ```bash
    docker rm {container id / 이름}
    ```
    
4. **Container 중지**
    
    ```bash
    docker stop {container id / 이름}
    ```
    
5. **중지된 Container 시작**
    
    ```bash
    docker start {container id / 이름}
    ```
    
6. **Container 재시작**
    
    ```bash
    docker restart {container id / 이름}
    ```
    

---

## Container Interaction Command
- 현재 실행중인 Container와 소통할 때 사용하는 명령어

1. **실행중인 Container 내부에서 새로운 프로세스 실행**
    
    ```bash
    docker exec {container id / name} {command}
    ```
    
2. **Container Log 확인**
    - -f/-follow: 실시간 log확인

    ```bash
    docker logs {option} {container id / name}
    ```
    
3. **Container 죽이기**
    - -s: 보낼 signal 정함
    - SIGTERM, SiGHUP, SIGINT 등의 signal이 있음

    ```bash
    docker kill {option} {container id / name}
    ```
    

---

## Debugging Command

1. **Docker 객체(컨테이너, 이미지, 볼륨, 네트워크 등)의 상세 정보 확인**
    - JSON 형식으로 반환하여 보여줌

    ```bash
    docker inspect {object id / name}
    ```
    
2. **실행중인 Container의 실시간 자원 사용량 및 통계량 확인**
    
    ```bash
    docker stats {container id / name}
    ```
    
3. **Container 내부의 Process 확인**
    
    ```bash
    docker top {container id / name}
    ```
    
4. **Docker Demon이 보내는 이벤트 확인**
    
    ```bash
    docker events
    ```

---

## Docker Volume Command

1. **Docker Volume 생성**
    
    ```bash
    docker volume create {volume 이름} # named volume 생성
    docker run -v {Volume을 mount할 위치} {image 이름} # anonymous volume 생성
    docker run -v {local의 위치}:{volume을 mount할 위치} {image 이름} # bind mounts 생성
    ```
    
2. **Docker Volume들 확인**
    
    ```bash
    docker volume ls
    ```
    
3. **Docker Volume에 대한 정보 확인**
    
    ```bash
    docker volume inspect {volume 이름}
    ```
    
4. **Docker Volume 삭제**
    
    ```bash
    docker volume rm {volume 이름}
    ```
    
5. **사용하지 않는 Volume 삭제**
    
    ```bash
    docker volume prune
    ```
    

---

## Docker Compose Command

1. **Compose 실행**
   - compose.yml파일을 기반으로 compose 실행
    
    ```bash
    docker-compose up
    # docker-compose up --build: 이미지 build후에 compose 실행
    ```
    
2. **Compose 중지**
    
    ```bash
    docker-compose down
    # -v 옵션을 주어 volume까지 삭제 가능
    ```
    
3. **Compose 실행 여부 확인**
    
    ```bash
    docker-compose ps
    ```
    
4. **Compose 로그 확인**
    
    ```bash
    docker-compose logs
    # -f 옵션으로 실시간 log확인 가능
    ```
    
5. **특정 Service만 재시작**
    
    ```bash
    docker-compose restart {service_name}
    ```