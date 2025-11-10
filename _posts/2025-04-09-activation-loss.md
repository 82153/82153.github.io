---
layout: article
title: "[DL Basic] 활성화 함수 & 손실 함수"
date: 2025-04-09 12:54 +0900
category: DL Basic
tags: [DL Basic]
sidebar:
  nav: dl_basic
---
## 활성화 함수(Activation Function)

- 활성화 함수는 가중 합의 결과를 일정 기준에 따라 값을 변화시키는 **비선형 함수**이다. 이 활성화 함수는 선형으로는 해결할 수 없는 즉, 비선형적으로 분류되는 문제를 해결할 수 있도록 **비선형성을 추가해주는 역할**이다.
- 대표적으로 **Sigmoid, Tanh, ReLU, Softmax** 등이 있다.

### Sigmoid = $\frac 1{1+e^{-x}}$

- 가중 합의 결과를 [0 ~ 1] 사이에서 비선형 형태로 바꿔준다.
    
    <img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/3846643a-0f9f-4a1a-a6be-eec73fe5f3f7" />

- 주로 로지스틱 회귀 같은 분류 문제에서 주로 사용했다.
- 하지만 모델의 깊이가 깊어지며 발생하는 **기울기 소멸 문제가 발생**하여 잘 사용하지 않는다. Sigmoid의 미분 값을 보면 값들이 [0 ~ 0.25] 사이이다. 이는 역전파 시에 기울기가 계속 줄어든다는 의미이고, 계속 곱해지다보면 기울기가 소멸하게 된다.
- 또한 함수의 값이 모두 양수(Non-zero centered)로 학습 시에 **모든 파라미터가 동일한 방향으로 학습하게 되어 불안정한 학습**을 하게 된다.
- 이와 같이 모든 가중치의 업데이트가 같은 방향으로 이뤄지는 문제를 **Zig-Zag문제**라고 한다.
    
    ![스크린샷 2025-04-18 220458](https://github.com/user-attachments/assets/b6e73b42-faa5-4690-a368-1de021bf6c6b)


### **하이퍼볼릭 탄젠트 = Tanh**

- 하이퍼볼릭 탄젠트는 가중 합의 결과를 [-1 ~ 1]사이에서 비선형 형태로 변형해준다.
    
    <img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/e5cf5541-5ed9-4ead-b4f8-41975e1bbe39" />


- **Sigmoid에서 양수만 나오는 문제를 해결**하였다. 하지만 여전히 기울기가 [0 ~ 1]사이로 딥러닝이 깊어지며 발생하는 **기울기 소멸 문제가 발생**한다.

### **ReLU = max(0, x)**

- 최근 가장 활발하게 사용되는 활성화 함수이다. input이 음수이면 0을 출력하고, 양수이면 x를 출력한다. 이는 경사하강법에 영향을 주지 않아 **학습 속도가 빠르고, 기울기 소멸 문제가 발생하지 않는다.**
    
    <img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/695969f0-05da-41c0-bd3a-91cf487ed022" />

- 하지만 음수는 항상 0으로 처리하여, 기울기가 0이 되는 정보가 손실되는 **Dying ReLU 문제**가 발생한다. 즉, 0보다 작은 값들이 많이 나온다면 파라미터 업데이트가 원활하지 못하다.
- 또한, ReLU에서도 모든 값이 0 이상이고, 기울기도 0 아님 1이 나온다. 즉, ReLU 또한 **Zig-Zag문제**를 겪을 수 있다.

### **Leaky ReLU = x if x > 0 else $\alpha$x**

- ReLU에서 음수를 항상 0을 출력하는 문제를 해결하기 위해 **음수 부분에 매우 작은 기울기**를 주는 방법이다. 음수 부분에 주는 기울기는 하이퍼 파라미터로 사전에 정의해줘야한다.
    
    <img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/b50e97c8-ae90-4964-8cee-06ebcb7fc1d8" />

- 이 음수 부분의 기울기($\alpha$)를 학습하는 방식으로 **PReLU**가 존재한다.

### Softmax = $\frac {exp(a)}{\sum_{i=1}^{n}{exp(a)}}$(n = 출력층의 개수)

- 소프트맥스는 입력값을 [0~1] 사이로 출력되도록 정규화하여 출력값들의 값이 항상 1이 된다.
- 주로 출력 노드의 활성화 함수로 많이 사용된다.

### **그 외의 활성화 함수**

- **ELU**: ReLU의 음수 영역 문제를 완화하기 위해 음수 부분에 지수함수를 적용한 활성화 함수이다.
- **GELU**: 입력값이 활성화될 확률을 정규분포 기반으로 반영하며, BERT, GPT 등에서 사용되었다.
- **SiLU(Swish)**: x * sigmoid(x) 형태의 부드러운 비선형 함수로, 깊은 네트워크에서 학습 안정성이 좋다.
- **Mish**: x * tanh(softplus(x)) 형태로, SiLU보다 음수 영역 곡률이 더 완만하여 표현력이 더 유연하다.

<table>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/744e6316-52ac-4d06-ad68-ccd240837b28" width="300" height="200" alt="image"></td>
    <td><img src="https://github.com/user-attachments/assets/78c73f3d-b5a6-483c-a126-daaf8f4cf8b9" width="300" height="200" alt="image"></td>
  </tr>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/f8897c69-4429-4334-8be3-a29f82b4c5a0" width="300" height="200" alt="image"></td>
    <td><img src="https://github.com/user-attachments/assets/8ce5d178-e9c7-4534-b099-8a3056bb9dd3" width="300" height="200" alt="image"></td>
  </tr>
</table>

---

## 손실 함수

- 손실 함수는 학습을 통해 얻은 데이터의 **추정치가 실제 데이터와 얼마나 차이가 나는지 평가하는 지표**로 **오차를 구하는 방법**이다. 이 값은 0에 가까울 수록 잘 추정한거고, 값이 클수록 많이 틀렸다는 거다.

### 평균 제곱 오차(MSE)

- **실제 값과 예측 값의 차이를 제곱하여 평균** 낸 것이다. 주로 **회귀에서 사용**된다.
    
    ![스크린샷 2025-04-18 231130](https://github.com/user-attachments/assets/f46aa4f8-a7ae-4593-9dc1-4df913927d56)

- 만약 제곱을 하지 않는다면, 차이를 더하는 과정에서 +, -로 인해 차이가 사라지게 되고, 제곱을 하므로써, 오차가 큰 거에 대해 더 큰 페널티를 줄 수 있다.

### 바이너리 크로스 엔트로피(Binary CrossEntropy)

- 바이너리 크로스 엔트로피는 **이진 분류**에서 사용하고, **레이블이 0과 1**로만 나온다.
    
    ![스크린샷 2025-04-18 232944](https://github.com/user-attachments/assets/fa6fbca8-27ed-458e-a5f9-83d11ca3ab80)


### 크로스 엔트로피(CrossEntropy)

- 크로스 엔트로피는 **분류 문제**에서 사용되고, **One-Hot Encoding**을 했을 때, 사용할 수 있다.
- 
    ![스크린샷 2025-04-18 232714](https://github.com/user-attachments/assets/05607a04-d054-40a2-b8c5-39b235aa7043)

- 이진 분류때처럼 1, 2, 3, .. 등으로 나타내지 않는 이유는 Class간에 관계가 생기기 때문이다. 즉, **오차가 Class마다 다르게 된다.**
- 크로스 엔트로피에 대한 자세한 내용은 [정보 엔트로피](https://82153.github.io/mathematics/2025/10/20/entropy.html)
