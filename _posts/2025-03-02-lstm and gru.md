---
layout: article
title: "[NLP] LSTM과 GRU"
date: 2025-03-02 18:38 +0900
category: NLP
tags: [NLP]
aside:
  toc: true
sidebar:
  nav: NLP
---
## LSTM(Long Short-Term Memory)

- RNN과 동일하게 Sequence 데이터를 처리하는 모델이며, 매 time step마다 동일한 모듈을 반복적으로 적용한다. 이 때, 4개의 Vector(gate)를 계산하는 신경망이 존재한다.
    
    ![image.png](attachment:af32f066-10ed-4539-b2a1-c218b5c8ab26:image.png)
    
- 또한 hidden state를 기억 소자로 볼 수 있다는 점에서 hidden state를 더 복잡하게 구성하여 RNN보다 더 오래 기억할 수 있게 하여 **장기 기억 문제를 해결**하였다..
- LSTM에서는 **cell state를 추가**하여 hidden state를 더 복잡하게 만들었고, hidden state의 모체가 되는 **cell state가 실질적으로 RNN에서 hidden state의 역할**을 한다.
- **LSTM의 gate**
    
    ![image.png](attachment:9e132175-3953-49ee-a483-73bb3c988e78:image.png)
    
    - $f$: Forget gate로서, 이전 Cell의 내용을 지울지 결정한다. 즉 **이전 Cell의 내용을 얼마나 유지할지 결정**하는 것이다.
    - $i$: Input gate로서, Cell의 내용을 적을지 결정한다. 즉, **현재 time step의 정보를 얼마나 저장할지 결정**한다.
    - $g$: Function output로서, Cell에 적을 내용을 결정한다. 즉, **Input gate를 통해 Cell에 추가될 내용**이다.
    - $o$: Output gate로서, Cell의 내용을 출력한다. **Cell을 기반으로 최종 Hidden state $h_t$를 결정**한다.
- 위에서 LSTM이 장기 기억 문제를 해결하였다고 말했는데 그 부분에 대해서 알아보자.
    
    ![image.png](attachment:3f7554b1-f8b8-4f2a-a12c-b7529549feab:image.png)
    
- 위 그림을 보면 $c_t$를 계산할 때, + 연산이 진행된다. 예를 들어 $Z=X+Y$일때, $Z$를 $X$와$Y$에 대하여 각각 편미분을 하여 역전파를 수행한다 하였을 때, 각 편미분의 값이 1이기에 **$Z$에 도달한 Gradient 값이 그대로 $f$⊙$c_{t-1}$와 $i$⊙$g$에 전달**될 것이다.
- 이를 통해, RNN에서 $W_{hh}$를 반복적으로 곱해줌으로 생기는 **정보 왜곡을 방지**하고, **거리가 먼 time step에서도 문제 없이 정보를 전달**할 수 있게 된다.

---

## GRU(Gated Recurrent Unit(GRU)

![image.png](attachment:76200ab6-9e52-458e-8bc8-54a8c33f7f78:image.png)

- **Cell state와 Hidden state를 하나의 Vector로 통합**한 모델이다.