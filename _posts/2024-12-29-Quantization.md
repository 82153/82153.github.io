---
layout: article
title: "[Lightweighting]Quantization"
date: 2024-12-28 16:01 +0900
category: Lightweighting_Optimization
tags: [Lightweighting & Optimization]
aside:
  toc: true
sidebar:
  nav: layouts
---
## Quantization(양자화)

- 숫자의 **정밀도(precision)**를 낮추는 최적화 및 경량화 기법
- 여기에서 정밀도는 의도한 숫자를 얼마나 정확하게 표현할 수 있는지를 나타내는 것이다. 정밀도가 높을수록 표현력은 올라가기에 오차는 줄어들지만, 계산 속도는 느려지고, 정밀도가 낮아지면 표현력은 떨어져 오차가 증가하지만 계산 속도는 올라간다.
- 위와 같은 특징을 바탕으로 양자화는 **오차를 최소화하는 낮은 정밀도** 표현을 찾는 것이 목적이다.

## Quantization Mapping

- 높은 Precision 데이터를 낮은 Precision으로  대응시키는 Quantization 계산식을 말한다.
- 양자화를 거친 후에 우리는 어떤 값들을 저장하고 있어야 할까?
    - 당연하게 우선 **양자화 방식**과 **Quantized Value**를 저장해야 한다. 이뿐만 아니라 우리는 **Scale factor(s)**와 **Zero-point(z)**를 저장해야 한다. 이 값들을 통해 우리는 양자화된 값을 얻을 수도 있고, 역양자화를 통해 복원할 수 있다.
    - Scale factor는 기울기 값이고, Zero-Point는 양자화 후의 0의 위치이다.
        
        ![스크린샷 2024-12-28 141056](https://github.com/user-attachments/assets/8b1c6c47-d4e5-489b-bcd1-0db39d505d54)
        
    - s와 z를 구한 뒤, 위의 수식을 통해서 Quantization을 할 수도, 원본으로 복원할 수도 있다.
- Quantization 기법에는 대표적으로 2가지 방법이 있는데 **Absmax Quantization**과 **Zero-point Quantization**이 있다.
- **Absmax Quantization**
    - Absmax Quantization는 ****0을 기준으로, 변환 후 **값들의 범위가 좌우 대칭**이 되도록 변환하는 기법으로 Tanh처럼 **대칭적인 분포를 가지고,** **0을 항상 0으로 보내는 것**이 효과적인 경우에 활용한다.
    - 이 기법은 우선 데이터에서 절댓값의 최댓값을 내가 변환하려는 자료형의 최댓값으로 매핑하고 그 값의 음수를 최솟값으로 매핑하여 그 것에 맞게 다른 값들을  Quantization한다.
    - 여기에서 s = $\frac{max|X|}{변환하려는 자료형의 최댓값}$ , z = 0 이 된다.(추가적으로 z는 0이기에 따로 저장 안해도 된다.)
- **Zero-Point Quantization**
    - Zero-Point Quantization은 0을 고려하지 않고 **전체 범위를 균일**하게 변환하는 기법으로 ReLU처럼 **데이터 분포가 비대칭적이거나 평균 값이 0이 아닌 경우**에 활용한다.
    - 이 기법은 데이터의 최댓값을 변환하려는 자료형의 최댓값으로 매핑하고, 데이터의 최솟값을 변환하려는 자료형의 최솟값으로 매핑한다.
    - 이후 s = $\frac{max(X)-min(X)}{변환하려는 자료형의 전체 거리}$, z = 변환하려는 자료형의 최솟값 -  $round(\frac{min(X)}{s})$로 s와 z를 구한다.
- **Clipping**
    - Outlier 값의 영향을 줄이기 위해 일정 범주를 넘어가면 같은 값으로 취급하는 기술이다.
