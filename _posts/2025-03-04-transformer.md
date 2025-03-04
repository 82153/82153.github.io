---
layout: article
title: "[NLP] Transformer"
date: 2025-03-04 14:25 +0900
category: NLP
tags: [NLP]
aside:
  toc: true
sidebar:
  nav: NLP
---
## Self-Attention

- self-attention은 Transformer에서 사용되는 모듈로 Query(q), Key(k), Value(v) 벡터를 사용한다. 입력을 Embedding하고 각 단어에서의 Embedding을 q, k, v로 변환하여 자기 자신에 대하여 Attention연산을 진행하는 것이다.

![스크린샷 2025-03-03 160726](https://github.com/user-attachments/assets/6250d57c-3267-461b-8e43-51c0940d9cd6)

- 현재 단어의 Embedding에 대해서 $W_Q$를 통해 q를 구하고, 현재 단어를 포함한 모든 Embedding에 대하여 $W_K$와 $W_V$를 통해 k와 v를 구한다. 이 Embedding은 첫 Encoder 층에만 필요하고 이후에는 이전 Encoder 층의 출력을 사용한다.
- 이 후 q와 모든 단어의 k를 내적 후에 softmax연산을 취하고 이 결과를 가중치로 v의 가중 평균을 구한다.
- Attention에서는 Decoder의 현재 단어의 $h_t$와 Encoder의 모든 단어의 $h$와 내적을 구하고 softmax연산을 취한 뒤, 그것을 가중치로 Encoder의 $h$의 가중 평균을 구한다.
- 이와 같은 과정을 Self-Attention과 비교해보면, Attention에서는 Encoder의 $h$를 k와 v로서 사용한 것이고, Decoder의 현재 단어의 $h_t$를 q로서 사용한 것이다.
- 즉 현재 **Query와 가장 유사한 Key의 Value를 활용**하게 된다.
- Self-Attention의 수식적으로 표현하면 $\sum_{i}\frac{exp(q \cdot k_i)}{\sum_{j}exp(q \cdot k_j)}v_i$, 혹은 $softmax(QK^T)V$로 나타낼 수 있다.
- 이러한 연산을 위해서 Q와 K는 같은 크기의 차원 $d_k$를 가져야 한다.

---

## Scaled Dot-Product Attention

- 위와 같은 Self-Attention에서 Q와 K의 차원인 $d_k$가 커지면 $q \cdot k$의 분산이 증가하여 Softmax 내 특정 값이 굉장히 커져 편향되는 분포를 가질 수 있다. 이는 역전파 시에 **Gradient가 매우 작아지는 결과**로 이어져 학습이 매우 느려진다.
- 이를 해결하기 위해 $d_k$의 차원만큼 출력 값의 분산 값을 줄인$softmax(\frac{QK^T}{d_k})V$로 구하고 이를 Scaled Dot-Product Attention이라 한다.

---

## Multi-Head Attention

- Self-Attention에서는 Query와 가장 유사한 Key의 Value를 활용하는데, 하나의 Softmax에 기반한 하나의 Attention을 사용한다. 이는 하나의 **단어가 단 하나의 다른 단어의 정보에만 집중**하게 되어 이 외의 다른 정보들은 적은 정보만을 포함할 수 있게 된다.
- 이러한 문제를 해결하기 위한 것이 **Multi-head Attention**이다. 말 그대로 여러 Self-Attention head들을 독립적으로 활용하여 다양한 정보를 포함할 수 있게 된다.

![스크린샷 2025-03-04 131051](https://github.com/user-attachments/assets/1e0f5a8b-2cf4-4827-aa54-a23ebf6d0838)

- 각 head에서 나온 값을 concat하고, 가중치 행렬 $W^o$를 곱하여 Attention의 최종 결과를 출력한다.
- 위의 내용을 바탕으로 -Attention의 과정은 아래 그림과 같다.

![스크린샷 2025-03-04 131351](https://github.com/user-attachments/assets/0cb7adbd-a806-480d-93a4-f3820b72795d)

---

## Transformer

- 기존의 Seq2Seq 모델과 달리 RNN모델을 사용하는 것이 아닌 Multi-Head Attention 모듈을 사용한 모델이다.

![스크린샷 2025-03-04 135046](https://github.com/user-attachments/assets/dae1aaac-c6a6-477c-99a1-79e404aaebec)

- Transformer의 Encoder 부분은 block이라 불리는 것으로 구성되고, 이 block이 여러 번 반복하고 첫 Block만이 Embedding 벡터를 입력으로 받고, 이후 Block들에서는 이전 Block의 Output을 입력으로 받는다.
- Transformer의 Block은 **Multi-Head Attention**과 **2층의 layer로 구성된 Feed Forward Network**인 sub-layer 구성된다.
- 또한 각 sub-layer는 **Residual Connection과 Layer Normalization**을 진행한다.

---

## Layer Normalization

![스크린샷 2025-03-04 141939](https://github.com/user-attachments/assets/411d39f6-45d1-4453-b848-9799ceff4488)

- Layer Normalization은 각 단어 벡터에 대하여 Normalization을 진행하는 것이다.
- 우선 각 단어 벡터에 대하여 Normalization을 통해 평균과 분산을 0과 1로 변환하고, 이 결과를 각 차원 별로 학습 가능한 파라미터로 Affine Transformation을 진행하여 최종 결과를 얻는다.

---

## Positional Encoding

- Transformer Block은 입력 순서가 바뀌어도 똑같은 출력을 낸다. 즉 **Transformer Block은 단어들의 순서를 고려하지 않는다**는 것이다. 이를 Permutation-invariant라고 한다.
- Transformer Block이 어순을 고려하도록 하기 위해 각 단어의 위치 정보를 넣을 필요가 있다. 원 논문에서는 **Sinusoidal function을 활용하였고, 학습 가능한 Positional Embedding을 사용**하기도 한다.

![스크린샷 2025-03-04 142904](https://github.com/user-attachments/assets/df5e7ee0-6be7-4c1b-ba6e-50366ee1ec6c)

- 위의 그림이 Sinusoidal function을 시각화한 것으로 $i$번째 입력에 대해 위와 같이 Encoding의 결과를 얻고 이것을 $i$번째 입력 토큰에 더해주는 식으로 적용된다.

---

## Masked Attention

- Transformer에서 추론 시에는 하나의 단어 씩 순차적으로 단어를 예측하는 Auto-Regressive한 방식이다.
- Decoder 부분에서도 모든 단어에 대해서 Self-Attention 연산을 하는데, 아직 **예측되지 않은 단어에 접근하지 못하게 하는 방법이 Masked Attention**이다.
    
    ![스크린샷 2025-03-04 152001](https://github.com/user-attachments/assets/fe2edfca-49d3-4077-9dea-5ba96d9a7d2f)

- 위 그림과 같이 Q와 K를 행렬 곱한 결과에서 아직 예측되지 않은 단어의 Query와 Key를 0으로 바꾸어 Softmax 연산 이후 V와 행렬 곱 연산을 할 때, 가중치를 0으로 하여 아직 예측되지 않은 단어를 참조하지 못하게 하는 방법이다.
- 추가적으로 학습 시에는 특히 초반 부는 Random Initialize된 파라미터에 가까울 터이니 이상한 예측값이 나올 것이다. 이를 방지하기 위해 **학습 시에는  추론할 때처럼 이전 Output을 Input으로 사용하는 것이 아닌 GT를 Input으로 넣는다.**
