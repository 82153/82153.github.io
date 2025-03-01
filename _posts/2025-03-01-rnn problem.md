---
layout: article
title: "[NLP] Tokenization"
date: 2025-03-01 19:38 +0900
category: NLP
tags: [NLP]
aside:
  toc: true
sidebar:
  nav: NLP
---
## Exploding and Vanishing Gradient of RNN

- RNN은 hidden-state($h_t$)에 이전 time step에 대한 정보를 가지고 있다. 하지만 이 **$h_t$는 고정된 크기의 벡터이기에 긴 time step에 대한 정보를 모두 저장하기에는 한계**가 있다.
- 이 부분을 좀 더 자세히 보기 위해 RNN의 학습 방법을 다시 보게 되면 아래의 그림과 같이 각 time step을 Forward 과정을 진행한다.
    
    ![image.png](attachment:abed4796-8214-4097-9c98-437ec9d10d31:image.png)
    
- 여기서 $\frac{dy_3}{dh_1}$을 구한다고 해보자. 그럼 $\frac{dy_3}{dh_3} \times \frac{dh_3}{dh_2} \times \frac{dh_2}{dh_1}$으로 구할 수 있을 것이고, tanh는 최대 1까지의 값을 가지니 이를 바탕으로 부등식으로 표현하면 $\frac{dy_3}{dh_1} ≤ d \times \frac{a}{2} \times \frac{a}{2}$로 나타낼 수 있다.($a = W_{hh}$)
- 지금은 단순히 time step의 차이가 2 밖에 나지 않지만, time step이 길어짐에 따라 $\frac{a}{2}$가 time step만큼 곱해져 Gradient의 값이 기하급수적으로 증가하거나 감소하게 된다.
- 만약 a가 1보다 크게 되면 **Gradient가 무한대로 되어 Exploding Gradient**문제가 발생하고, **a가 1보다 작다면 Gradient가 0에 수렴하여 Vanishing Gradient**문제가 발생하게 된다.
- 이를 해결하기 위해 **Gradient의 절대값이 Threshold 이상이면 Threshold 값으로 바꾸는 Gradient Clipping** 같은 기법도 있지만 완전한 해결 방법은 아니다.