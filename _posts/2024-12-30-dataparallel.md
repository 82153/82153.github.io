---
layout: article
title: "[Distributed Training] Data Parallelism"
date: 2024-12-30 15:21 +0900
category: Lightweighting_Optimization
tags: [Lightweighting & Optimization]
aside:
  toc: true
sidebar:
  nav: lightweighting
---
## Distributed Training(분산 학습)

- **여러 GPU 간에 데이터를 분할하거나 모델 자체를 분할하여 병렬화 하는 학습 기법**으로 GPU에 한 번에 들어가지 않는 큰 모델도 학습 가능하도록 하고 최근 모델의 크기가 굉장히 커짐에 따라 필요성이 증가했다.
- 데이터를 분할하여 처리하는 **Data Parallelism**과 모델을 분할하여 처리하는 **Model Parallelism**이 있다.

## Data Parallelism

- **큰 데이터를 여러 GPU들에 분할**하여 동시에 처리함으로써 학습 속도를 높이는 기법이다.
    
    ![스크린샷 2024-12-30 141903.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6acf52cb-2c5c-462e-bf26-b7011fa2da60/57feeb85-bc4b-40b4-8b25-7046c3404ce1/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-12-30_141903.png)
    
- 우선 모델을 모든 GPU에 복제하고, 데이터셋을 minibatch 단위로 나누어 각각 할당한다.
- 이후, 각 GPU들에서 데이터에 대한 연산을 병렬적으로 수행하고, 각 GPU에서 계산된 Gradient를 하나의 Gradient로 합쳐, 마스터 노드가 병합 및 모델의 Parameter를 업데이트한다.
- Data Parallelism은 **DP(DataParallel)**와 **DDP(DistributedDataParallel)**로 나뉜다.
- **DP(DataParallel)**
    - DP는 우선 마스터 노드가 GPU 개수에 맞게 데이터를 나눠주고, 또한 동일한 weights들을 각 GPU에 전달한다.
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6acf52cb-2c5c-462e-bf26-b7011fa2da60/f1feedbc-88a8-4c93-b4b4-89f3762efb0c/image.png)
        
    - 이를 통해 각 GPU에서 Forward Pass를 진행하고, 마스터 노드에서 각 GPU의 Logit 값을 취합하여 Loss를 계산한다.
    - 이렇게 계산된 Loss는 각 GPU에 전송되고 그 Loss에 맞게 각 GPU에서 Gradient가 업데이트된다. 이렇게 업데이트 된 Gradient값을 마스터 노드가 다시 취합하여 마스터 노드에서 모델 weight를 다시 업데이트 한다.
    - 마지막으로 마스터 노드의 업데이트 된 weight를 각 GPU에 동기화 한다.
    - 이 기법은 마스터 노드가 파라미터 전송, logit 취합 등 혼자 많은 일을 하기에 마스터 노드에서 GPU 부하(병목)이 발생한다는 단점이 있다.
- **DDP(DistributedDataParallel)**
    - 위의 DP 방식에서 마스터 노드만 너무 일하는 것을 막기 위해 **마스터 노드를 없애고 모델 업데이트에 필요한 값들은 공유하고, 업데이트는 각자 하는 방식**이다.
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6acf52cb-2c5c-462e-bf26-b7011fa2da60/79ba08f7-c1e9-44ca-b7c9-512e40cba6c5/image.png)
    
    - 필요한 값을 공유하기 위해 **AllReduce Operation**을 사용하게 된다.
    - AllReduce Operation은 **여러 디바이스에 흩어져 있는 데이터를 서로 동시에 주고받기 위한** Collective Operation 중 하나로 **여러 디바이스의 데이터를 모두 모아 하나의 값으로 줄인 후 그 결과를 디바이스에 전송하는 방식**이다.
    - 우선 모델을 각각의 GPU에 복제하고 Foward과정을 거친다. 이후 따로 취합 없이 독립적으로 Logit값을 계산하여 Loss를 구하고 각각 Gradient를 구한다.
    - 이후 각 GPU의 Gradient값을 AllReduce Operation을 통해 평균 Gradient를 구하고, 이 평균 Gradient를 기반으로 모든 GPU를 업데이트 한다.