---
layout: article
title: "[NLP] Seq2Seq with Attention"
date: 2025-03-03 13:57 +0900
category: NLP
tags: [NLP]
aside:
  toc: true
sidebar:
  nav: NLP
---
## Seq2Seq Model의 문제점

![image.png](attachment:4175e5bd-6599-4142-8f18-8ea553734b33:image.png)

- Seq2Seq 모델은 many-to-many를 다룰 수 있는 기본 구조의 모델로서, 일련의 **단어를 Encoder가 입력으로 받아 고정된 크기의 Latent Vector(=마지막 time step의 $h_t$)로 변환**하고 **이를 Decoder의 $h_0$로 할당하고 SOS토큰을 입력으로 주어 단어를 순차적으로 예측**한다.
- 이런 모델은 **가변적인 길이를 가진 input과 output을 다룰 수 있게 된다.**
- 하지만 Latent Vector가 입력으로 들어온 문장의 모든 정보를 잘 담고 있어야 하지만 **Latent Vector가 고정된 크기이기에 긴 입력 문장에 대해서 모든 정보를 담기 힘든 Bottleneck 문제가 발생**한다.

---

## Seq2Seq with Attention

- Seq2Seq모델의 bottleneck 문제를 해결하기 위해 고안된 방법이 **Attention**이다.
- Attention은 Encoder의 모든 time step의 $h_t$에 대하여, Decoder의 각 time step마다 필요한 정보(Encoder의 $h_t$)를 선택하여 사용하는 방식이다.
- 이를 통해, Decoder는 Encoder의 마지막 $h_t$에만 의존하지 않고 **Encoder 상의 모든 $h_t$를 활용하여 Decoding 과정에서 필요한 정보를 선택하여 사용**하는 것이다.

![image.png](attachment:61595e6f-a4ea-48a3-ade7-d466f5aa8c08:image.png)

- Attention의 과정을 살펴보면, 우선 **Decoder의 현재 time step의 $h_t$와 Encoder의 각 time step의 $h$간의 내적이나 concat 후에 MLP를 거치는 등의 방법을 통하여 Attention Score(유사도)**를 구한다.
- 이렇게 얻어진 **Attention Score에 Softmax를 적용**하여 유사도의 확률 분포를 얻고, 이 값들을 **각 Encoder의 $h$의 가중치로 사용하여 가중 평균을 얻어 최종 Attention 출력**을 생성한다.
- 이 **Attention 출력과 해당 time step의 Decoder의 $h_t$와 concat하여 Decoder의 output을 생성하는 Layer에 입력**으로 주어 최종 예측을 생성한다.
- Attention 모듈을 추가하여 Seq2Seq모델은 다양한 장점을 얻었다.
    1. Decoder가 입력의 특정 부분들에 집중할 수 있게 하여 **기계 번역의 성능이 크게 향상**되었다.
    2. Decoder가 Bottleneck을 우회하여 입력을 직접 확인할 수 있게 되어 **Bottleneck 문제를 해결**했다.
    3. 아주 먼 과거 $h_t$까지 가는 Gradient flow의 지름길을 제공하여 **Vanishing Gradient 문제가 발생하지 않게 되었다.**
    4. Attention score를 시각화한 Attention map을 통해 Decoder가 어디에 집중했는지 알 수 있기에 **사람이 해석 가능**해졌다. 즉 **서로 다른 언어 간에 단어 레벨의 Alignment를 학습한 것**으로 명시적으로 따로 학습할 필요가 ㅇ벗다.