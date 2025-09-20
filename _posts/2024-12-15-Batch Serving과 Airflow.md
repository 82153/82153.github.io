---
layout: article
title: "[Airflow]Batch Serving과 Airflow"
date: 2024-12-15 10:37 +0900
category: Airflow
tags: [Airflow]
aside:
  toc: true
sidebar:
  nav: Airflow
---
## Batch Serving

- 일정 기간 데이터 수집 후 **일괄 학습 및 결과 제공**을 하기에 **대량의 데이터 처리 시 효율적**이다.
- Batch Processing : 예약된 시간에 자동으로 실행되는 소프트웨어 프로그램을 자동으로 실행하는 방법
- Batch Processing이 Batch Serving보다 더 큰 개념이다.

## Airflow

- Airflow 이전에는 Crontab을 사용했음
- Crontab에서 Cron 표현식을 사용하여 스케줄링한다.
- *****로 각각 분, 시간, 요일, 달, 요일을 지정한다.
- crontab의 문제점
    - 재실행 및 알림에 대해서 별도의 처리를 하지 않는다.
    - 에러가 발생한 경우 별도의 알림이 없어 직접 log를 확인해야 한다.
    - 하지만 이 log도 보기 어렵다.
    - 여러 파일을 실행하거나, 복잡한 파이프라인을 만들기 힘들다.
- Airflow의 핵심 개념
    - **DAGs**(Directed Acyclic Graphs)
        - Airflow에서 작업을 정의하는 방법, 작업의 흐름과 순서 정의
    - **Operator**
        - Airflow의 작업 유형을 나타내는 클래스
        - BashOperator, PythonOperator, SQLOperator 등 다양한 Operator 존재
    - **Scheduler**
        - DAGs를 보며 현재 실행해야 하는지 스케줄 확인
        - DAGS의 실행을 관리하고 스케줄링한다
    - **Executor**
        - 작업이 실행되는 환경
        - LoacalExecutor, CeleryExecutor 등 다양한 Executor가 존재

## Airflow Architecture
![스크린샷 2024-12-15 222543](https://github.com/user-attachments/assets/eaf04102-abee-4aa8-a63c-1844fddb26ac)

- **Scheduler**
    - 각종 메타 정보의 기록을 담당한다.
    - DAG 디렉토리 내 py파일에서 DAG을 파싱하여 DB에 저장 및 DAG들의 스케줄링 관리 및 담당을 한다.
    - 또한 Executor를 통해 스케줄링 기간이 된 DAG을 실행하고, 실행 진행 상황과 결과를 DB에 저장한다.
    - Executor는 스케줄링 기간이 된 DAG을 실행하는 객체이다.
- **Workers**
    - DAG의 작업을 수행한다.
    - Scheduler에 의해 생기고 실행되며, DAG run을 실행할 때 생기는 로그를 저장한다.
- **Metadata Database**
    - 메타 정보를 저장하는 공간이다.
    - Scheduler에 의해 Metadata가 저장된다.
    - 이 Metadata에는 파싱한 DAG 정보, DAG Run 상태와 실행 내용, Task 정보 등이 저장된다.
- **Webserver**
    - WEB UI를 담당한다.
    - Metadata DB와 통신하며 유저에게 필요한 메타 데이터를 웹 브라우저에 보여준다.
