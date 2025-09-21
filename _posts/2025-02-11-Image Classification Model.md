---
layout: article
title: "[CV] Image Classification Model 정리"
date: 2025-02-11 14:59 +0900
category: CV
tags: [CV]
aside:
  toc: true
sidebar:
  nav: CV
---
## LeNet-5

<img width="600" height="200" alt="스크린샷 2025-09-21 141707" src="https://github.com/user-attachments/assets/f04f5cb2-5e12-4dcd-b967-3c13a68fb371" />

- LeNet-5는 이미지 분류 문제를 성공적으로 해결한 최초의 CNN 모델로, Conv → Pool → Conv → Pool → FC → FC로 이루어진 간단한 모델이다.
- Conv Layer는 5x5 필터에 Stride를 1을 사용했고, Pooling Layer에서는 Stride 2에 2 x 2 Average Pooling을 사용했다.

---

## AlexNet

<img width="600" height="200" alt="스크린샷 2025-09-21 142425" src="https://github.com/user-attachments/assets/cbe37976-67e6-4a7c-9c2b-9abeb34bd9e6" />

- 5개의 Conv Layer, 3개의 Max Pooling Layer, 3개의 FC Layer로 구성된 LeNet-5와 유사하지만, 더 확장하면서 ReLU, DropOut 등이 적용된 모델이다.
- AlexNet은 학습 시에 총 **2개의 GPU**를 병렬적으로 학습하였다. 위쪽 Layer에서는 주로 Color와 상관 없는 특징을 학습하고, 아래 Layer에서는 Color와 관련된 특징을 학습하게 하였다. 또한 Gradient Vanishing을 방지하기 위해, **ReLU**를 사용하였고, **DropOut**과 같은 정규화 기법을 사용하였다.
- 추가적으로 **LRN(Local Response Normalization)**을 사용하였다. LRN은 같은 위치(x,y)에 있는 채널들 간의 활성값을 정규화하는 기법이다.  이 기법은 ReLU를 사용했기에 양수 부분에서 입력을 그대로 사용하기에 **극단적으로 큰 값들이 주변 값들에 억제하는 것을 방지**하기 위해 사용하였다. 이를 통해 일반화 성능을 향상 시킬수 있었지만, 최근에는 배치 정규화나 Layer 정규화를 더 많이 사용한다.

---

## VGGNet

<img width="300" height="500" alt="스크린샷 2025-09-21 145006" src="https://github.com/user-attachments/assets/2040ac12-b820-47da-80c6-d8a1b0e09579" />

- VGGNet은 3x3 Conv를사용하여  기존의 CNN 모델보다 Layer를 더 깊게 쌓고자한 시도를 한 모델로, 주로 VGG-16, VGG-19를 주로 사용한다.
- Layer가 깊어질 수록 Receptive Field가 넓어지기에, 더 좋은 성능을 기대할 수 있다.  하지만 Layer가 깊어진다는 것은 파라미터가 더 증가한다는 것이고 이는 더 많은 resource가 필요하다 단점을 유발한다.
- 이를 해결하기 위해 큰 Filter를 사용하기보다 작은 사이즈의 Filter를 여러 번 사용하는 방식을 사용했다. VGGNet에서는 **모든 Conv Layer가 3x3 Filter**를 사용한다.
- 3x3 Filter만을 사용함으로써, **더 적은 파라미터로 학습**이 가능해지고 더 **효율적으로 Receptive Filed를 확장**할 수 있었고, **더 많은 비선형성을 부여**하여 더 복잡한 문제를 풀기에 용이해졌다.
- 주로 16 Layer나 19 Layer를 사용한다 하였는데 더 깊게 쌓지 않은 것은 **Gradient Vanishing/Exploding**과 **OverFitting**문제 때문에 더 깊게 쌓지 못 하였다.

---

## ResNet

<img width="600" height="400" alt="스크린샷 2025-09-21 151058" src="https://github.com/user-attachments/assets/59e80db7-f960-420f-85e9-09d34868b545" />

- ResNet은 이전의 VGGNet에서 19 Layer까지 밖에 못 쌓았던 한계를 Residual Connection을 통해 해결한 모델이다.
- VGGNet에서 더 깊이 못 쌓은 원인 중에 OverFitting문제를 지적했지만 ResNet에서는 이는 OverFitting이 문제가 아니라고 말한다.

<img width="400" height="300" alt="스크린샷 2025-09-21 151641" src="https://github.com/user-attachments/assets/dce1555b-9ad0-47c2-8198-b5a7d6305db2" />

- 위 그림을 보면 test error를 보면 층이 더 깊은 모델이 error가 더 크지만, training error 또한 더 깊은 모델이 더 크다. 이는 우리가 일반적으로 아는 OverFitting에서 나타나는 현상과 다르다.
- 이는 **Degradation Problem**으로 학습 난이도가 올라가서 학습이 잘 안되는 **Optimization 문제**라는 것이다.

<img width="400" height="250" alt="스크린샷 2025-09-21 152536" src="https://github.com/user-attachments/assets/b79d18a3-32fb-4d52-9721-2826f5627a1e" />

- 이를 해결하기 위해 ResNet에서 도입한 것이 **Residual Connection**이다. Residual Connection은 단순하게 Conv Block의 Output($F(x)$)에 Input($x$)를 더한 것이다.
- $H(x) = F(x) + x$를 Conv Block의 Output으로 사용하므로써, Weight Layer가 학습시 사용하게 되는 Gradient는 F(x)에서 오는 것으로 이는 $H(x) - x$로서 단순히 **Input과 Output의 차이(Residual)만을 학습**하게 되어 더 단순하게 학습할 수 있게 되어 Degradation Problem을 해결할 수 있었다.
- 또한, $H(x)$에 대한 Gradient($\frac{\partial H}{\partial x}$)를 계산할 때, $F(x)+x$를 미분하면 $\frac{\partial F}{\partial x} + \frac{\partial x}{\partial x}$가 된다. 여기서 $\frac{\partial x}{\partial x}$는 **1**이 되므로, Gradient가 1이 추가된 형태로 흐르게 되어,  **Gradient가 0으로 소실되는 것을 방지**하고, 깊은 층까지도 Gradient가 안정적으로 전달되어 학습이 원활하게 이루지게 된다.
- 추가적으로 Feature Map이 절반으로 줄어들 때마다, Channel을 2배로 키워 정보를 유지해준다.

---

## EfficientNet

<img width="600" height="400" alt="스크린샷 2025-09-21 154601" src="https://github.com/user-attachments/assets/79feeca6-5964-48e5-a139-b67f2ce7f850" />

- 기존 연구들에서는 Depth(layer)를 더 깊게 쌓는 방식으로 성능 향상을 이뤄냈다. EfficientNet은 Depth뿐만 아니라 Channel, Input의 Resolution의 크기를 키워 성능 향상을 이루려했다.
- 단순히 무작정 키우는 것이 아니라, **Compound Scaling**이라는 방식을 통해 세 요소의 **최적의 균형 조합**을 찾아, 제한된 Resource 내에서 Depth, Channel, Resolution의 최적의 조합을 찾아 제한된 Resource 내에서 이전 모델들 대비 좋은 성능을 이뤄냈다.

---

## ViT(Vision Transformer)

<img width="600" height="400" alt="스크린샷 2025-09-21 155504" src="https://github.com/user-attachments/assets/0945c349-c0e6-407a-ad53-0934ce583481" />

- ViT는 NLP분야에서 성공을 거둔 Transformer모델을 이미지 분류에 적용한 모델이다. 이미지를 동일한 크기의 **Patch 단위**로 나눈 뒤, 각 Patch를 **Linear Projection**을 통해 토큰 형태로 임베딩한다. (Patch = NLP에서의 단어)
- 이후에 **CLS 토큰까지 추가**해준다. 이 CLS 토큰은 전체 이미지를 대표하는 요약 정보를 학습하도록 한다.
- 각 이미지들의 위치를 반영하기 위해 **Positional Embedding을 추가**해준다. Transformer에서는 Sin/Cos함수 기반의 고정 Embedding을 하였지만, ViT에서는 **1D의 학습 가능한 Embedding**을 사용하였다.
- 이후 Transformer의 **Encoder 블록만 활용**하여 패치를 인코딩하고, CLS 토큰을 **MLP Head**에 통과시켜 최종적으로 이미지 분류를 수행한다.

---

## Swin Transformer

<img width="600" height="400" alt="스크린샷 2025-09-21 161833" src="https://github.com/user-attachments/assets/5f243e91-4a62-43b7-a6ac-284810dca5a2" />

- ViT는 이미지 전체에 대해서 Attention 연산을 수행한다. 이는 토큰 개수(N)일 때, 연산량이 $O(N^{2})$으로 **연산량이 굉장히 크다.** 또한 Flatten된 Patch 토큰을 Attention에 넣기 때문에, 이런 **계층적 구조가 부족**하여 ****작은 객체나 지역적 특징을 잘 포착하지 못한다.
- Swin Transformer는 이러한 단점을 극복하기 위해 **윈도우(Window) 기반 Attention**을 도입했다. 즉, 각 윈도우 내에서만 Attention을 수행하여 연산량을 줄이고, 다음 레이어에서는 윈도우를 **Shift**하여 서로 다른 영역 간의 상호작용도 가능하게 한다.
- 더 나아가, **Patch 병합**을 통해 계층적 구조를 형성합니다. 해상도를 줄이고 채널 수를 늘려가면서, CNN처럼 점진적으로 **지역적 특징에서 전역적 구조로 확장되는 표현 학습**이 가능해진다.
