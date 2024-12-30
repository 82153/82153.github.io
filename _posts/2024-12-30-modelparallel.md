---
layout: article
title: "[Distributed Training] Model Parallelism"
date: 2024-12-30 16:55 +0900
category: Lightweighting_Optimization
tags: [Lightweighting & Optimization]
aside:
  toc: true
sidebar:
  nav: lightweighting
---
## Model Parallelism

- **큰 모델을 여러 GPU들에 분할**함으로써 하나의 GPU로는 처리할 수 없는 대형 모델도 처리 가능하게 하는 기법이다.
- 모델을 어떻게 나누는 지에 따라 **Tensor Parallelism**과 **Pipeline Parallelism**으로 나뉜다.

|  |  Tensor Parallelism | Pipeline Parallelism |
| --- | --- | --- |
| **전략** | 모델의 텐서(weights)를 여러 GPU에 분산하여 각 GPU가 모델의 일부만 처리 | 모델의 여러 층들을 GPU에 분산하여 각GPU가 순차적으로 모델 일부를 처리 |
| **장점** | GPU당 메모리 사용량 감소 | 매우 큰 모델 처리가 가능하고, 메모리 부담 감소 |
| **단점** | GPU간 통신이 증가하여 병목(bottleneck) 가능 | 모델 층 간 데이터 전달로 인한 지연 발생 |
| **사용전략** | GPT-3와 같은 초대형 모델 학습에 적합 | 깊은 구조의 대형 모델에 적합 |

## Tensor Parallelism

- 텐서 연산을 나누어 수행하더라도 똑같은 값을 얻을 수 있다는 개념에서 시작한 것으로 **텐서 연산을 여러 차원의 슬라이스로 나눈 후 GPU들에 할당하여 처리하는 기법**이다.

![스크린샷 2024-12-30 154907](https://github.com/user-attachments/assets/9bffcbad-52fc-4ffe-b7df-099f4624a87d)

- 텐서를 나누는 방식에도 세로로 나누는 **Column-wise** 방식과 가로로 나누는 **Row-wise** 방식이 있다.
- 여기에서 Row-wise 방식으로 한다면, 행렬 곱의 조건이 맞지 않아 계산이 안 되는데 어떻게 처리할까?
    - 앞의 행렬(input)을 Column-wise한 방식처럼 세로로 나눠준다.
- Row-Wise와 Column-Wise 방식을 예시로 보면 아래와 같다.
    
    ![스크린샷 2024-12-30 160332](https://github.com/user-attachments/assets/7dba5975-2e58-4e98-89e6-a229c0ecff4d)
    
    ![스크린샷 2024-12-30 160349](https://github.com/user-attachments/assets/6b5055dd-9613-4018-bffd-ca26d0066e6d)
    
- Row-Wise에서는 X 또한 나누어 주는 것을 확인할 수 있고, 각각의 다른 GPU에서 계산하여 그것을 더해주는 작업을 한다. 여기에서는 합산 시에 AllReduce 연산을 통해 출력값을 동일하게 해준 뒤 Backward를 진행한다.
- Column-Wise에서는 나누어서 계산한 것들을 Column-wise하게 concat해주는 것을 알 수 있다. 또한 Backward 시에 AllReduce연산을 통해 열 단위의 Gradient를 모든 GPU에 동일하게 공유한다.

## Pipeline Parallelism

- **모델을 층 단위로 GPU에 분할**하여 순차적으로 넣어 주는 기법으로 계산 효율성을 높이고 메모리 사용량을 분산 할 수 있다.

![스크린샷 2024-12-30 163642](https://github.com/user-attachments/assets/3d4e6c7c-385d-4db1-9b06-fbc6a28f23c6)

- 하지만 층 간의 연산량 차이가 클 경우 Latency가 길어질 수 있다.
- 좀 더 자세히 설명하면 각 층은 서로 독립적인 것이 아니라 이전 층의 결과에 의존한다. 이렇기에 **이전 층의 결과가 나와야지만 다음 층의 계산을 할 수 있다**.
- 즉 Pipeline Bubble이라는 유휴 시간이 생기게 된다. 이를 최대한 줄이기 위해 우리는 **Batch를 좀 더 작게 나눈 Micro Batch를 사용하여 이전 층의 연산을 일찍 끝낼 수 있도록** 하여 Pipeline Bubble을 줄인다.
- 이 방법은 통신 횟수가 Micro Batch로 나눈 횟수에 비례하여 증가하기에 적당하게 나눠야 한다.
- 이 방법에도 Backward를 언제 처리하는 지에 따라 2가지로 나뉜다.
    - **Synchronous Pipeline**
        
        ![스크린샷 2024-12-30 163714](https://github.com/user-attachments/assets/cacb7539-c299-45b8-9c2b-3760b31120ea)
        
        - Forward 연산을 마치기 전까지 Backward 연산을 실행하지 않고, 모든 Backward를 마치면 한번에 모델을 업데이트 한다.
    - **Asynchronous Pipeline**
        
        ![스크린샷 2024-12-30 163738](https://github.com/user-attachments/assets/95af3d71-32f1-4b50-a75d-f98981929cef)
        
        - 하나의 Micro Batch의  Foward 연산과 Backward연산을 번갈아가면서 수행하는 방식으로, 위 방식보다 조금 더 유휴 시간을 줄일 수 있지만 구현이 어렵다.
        - 또한 파라미터를 모든 GPU가 공유하지 않기에 통신 비용이 발생하지 않는다.
        
        | **분류** | **Synchronous Pipeline** |  **Asynchronous Pipeline** |
        | --- | --- | --- |
        | **학습 속도** | 학습속도가 크게 증가하지만 동기화 과정에서 병목현상이 발생할 수 있음 | 동기화과정이 없으므로 학습 속도 크게 증가 |
        | **메모리 효율성** | 모든 GPU에서 gradients들이 동기화되어야 하기 때문에 메모리 효율성이 상대적으로 낮음 | 동기화 과정에서 생기는 idle time을 최소화하기 때문에 메모리 효율성이 높음 |
        | **수렴도** | 안정적이고 모델의 정확도가 높음 | 각 GPU에서 비동기적으로 gradients를 업데이트하기 때문에 불안정하고, 정확도 상대적으로 낮음 |
        | **장점** | 구현이 상대적으로 쉬움 | 학습 속도, 메모리 효율성 증가 |
        | **단점** | Gradient 동기화 과정으로 인한 유휴 시간 발생 | 구현이 매우 복잡 |
        | **사용 전략** | 동일한 GPU를 여러 개 사용하고 모델이 안정적으로 수렴해야 할 때 사용 | GPU 환경이 각각 다르고, 메모리 효율성을 최대화할 때 사용 |
- Pipeline Parallelism은 속도, 메모리 효율성, 수렴 측면에서 Trade-off가 존재한다.
    - Micro Batch 개수 $\uparrow$ → 수렴도, 메모리 효율성 $\downarrow$, 유휴 시간 $\uparrow$
    - 파이프 라인 스테이지 개수 $\downarrow$ → 유휴 시간 $\uparrow$
