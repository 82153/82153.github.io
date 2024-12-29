---
layout: article
title: "[PEFT] Advanced PEFT"
date: 2024-12-29 14:44 +0900
category: Lightweighting_Optimization
tags: [Lightweighting & Optimization]
aside:
  toc: true
sidebar:
  nav: lightweighting
---
## AdapterFusion

- Adapter 모듈은 특정 Task 1개에 대해서만 풀 수 있기에, 다른 Task를 풀고자 할 때에는 다른 Adapter 모듈로 교체해줘야 한다.
- 위와 같은 번거로운 작업을 하지 않기 위해 **Adapter를 결합하여 여러 Task를 해결하는 모델을 만드는 것**이 AdapterFusion이다.

![스크린샷 2024-12-29 142006](https://github.com/user-attachments/assets/7f69377f-b723-4500-8dd2-223d55e346d1)

- AdapterFusion은 **Knowledge Extraction**, **Knowledge Composition** 과정으로 이뤄진다. Knowledge Extraction 과정은 단순히 각각의 Adapter를 학습 시키는 과정이고, Knowledge Composition은 Adapter들의 결과를 가중합으로 합치는 과정이다.
- Knowledge Composition을 자세히 보면
    1. **병렬 Adapter 연산**
        - 각 Adapter의 출력을 병렬적으로 구하는 과정이다.
    2. Adapter-Attention
        - 사용자 입력에 맞는 Adapter의 가중치를 측정하는 Attention 모듈만 재학습한다.
        - Attention 모듈에서 Q는 사용자 입력의 Hidden state, V는 각 Adapter의 출력, K는 각 Adapter의 Key이다. 이 Q, K, V가 학습하는 파라미터이다.
        - 학습한 Q, K, V를 통해 Attention 모듈을 통해, 각 Adapter 출력의 가중합을 얻을 수 있다.

## QLoRA

- **LoRA에 Quantization 기법을 추가**로 활용하여 더 효율적인 방법이다.

![스크린샷 2024-12-29 144028](https://github.com/user-attachments/assets/e61c49ea-303e-433f-8a4b-4a440b8573c9)

- QLoRA는 Quantization을 2번 사용을 하는데
    1. **사전 학습된 Weight**들에 대해서 Quantization
        - 이 때, 원본 값 복원을 위한 양자화 상수(Scale factor, Zero-Point)가 발생한다.
    2. 1번에서 발생한 **양자화 상수**에 대해서 Quantization
        - 1번에서 발생한 양자화 상수는 중요한 값이기에 높은 Precision으로 저장된다.
        - 이 양자화 상수를 저장하는 비용을 조금이라도 절약하기 위해 한번 더 Quantization을 진행하게 된다.
