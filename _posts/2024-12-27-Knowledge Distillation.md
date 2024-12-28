---
layout: article
title: "Knowledge Distillation"
date: 2024-12-27 13:25 +0900
category: Lightweighting_Optimization
tags: [Lightweighting & Optimization, Knowledge Distillation]
aside:
  toc: true
sidebar:
  nav: layouts
---
## Knowledge Distillation(지식 증류)

- 경량화의 한 종류로서, 고성능의 Teacher 모델로부터 지식을 전달 받아 Student 모델을 학습 시키는 기법이다.

| **모델** | **성능** | **속도** | **파라미터 수** | **학습** |
| --- | --- | --- | --- | --- |
| Teacher | 좋음 | 느림 | 많음 | 사전 학습 |
| Student | 보통(상대적으로 낮음) | 빠름 | 적음 | 학습 대상 |

## KD 분류 체계

- KD 분류 체계는 **Knowledge**와 **Transparency** 관점으로 나눌 수 있다.
    - **Knowledge**: 증류할 지식의 따른 분류이다.
        - **Response-based**
            - **Logit-based**: Teacher의 Logit값을 사용한다.
            - **Output-based**: Teacher의 Output을 사용한다.
        - **Feature-based**: Teacher의 중간 layer의 **feature/representation**을 사용한다.
    - **Transparency**: 모델 내부 구조/파라미터 열람 가능 여부에 따른 분류이다. 즉 접근 권한이 어느 정도인지를 나타낸다.
        - **White-box**: Teacher의 **내부 구조, 파라미터** 등을 알 수 있는 경우이다.
        
        ![스크린샷 2024-12-27 103129](https://github.com/user-attachments/assets/63a7960e-ff07-4c25-87f7-d46393971801)
        
        - **Gray-box**: Teacher의 **output 및 최종 Logit / 제한된 정보**를 알 수 있는 경우이다.
        
        ![스크린샷 2024-12-27 103115](https://github.com/user-attachments/assets/9a384d46-37af-4d10-8056-71c4430c285c)

        - **Black-box**: Teacher의 **output만** 알 수 있는 경우이다.
        
        ![스크린샷 2024-12-27 103100](https://github.com/user-attachments/assets/4f5c52be-ad17-4a2d-b15f-c0a930bcb804)
        

## Logit-based KD

- Class 간의 유사도 정보가 간접적으로 있기에 Logit으로 계산된 **class 확률값**을 지식 증류에 사용하는 기법이다.
    
    ![스크린샷 2024-12-27 110147](https://github.com/user-attachments/assets/14c0c54f-60bf-4736-99cd-271f94e3a468)
    
- Student 모델의 학습은 Teacher모델의 확률 분포와 Student 모델의 확률 분포가 비슷해지게 학습을 진행한다. 이때, KL-Divergence로 비교하고 이것을 Loss로 사용한다.
- 정리해보면, Teacher 모델은 **Hard Label(정답 Class만)**으로 학습하고, Student 모델은 **Soft Label(Class별 확률)과 Hard Label**로 학습한다.
    
    ![스크린샷 2024-12-27 123427](https://github.com/user-attachments/assets/c70e05f4-ddcf-4f92-ad9f-d992258db4d2)
    
- Class 확률 값을 구할 때, $\tau$로 나눠 주는데, **$\tau$가 1보다 크면 확률 분포가 완만**해지고, **$\tau$가 1보다 작으면 확률 분포가 날카로워** 진다.
- 확률 분포를 완만하게 하는 것이 Class간의 유사도를 더 잘 파악할 수 있고, 또한 Label Smoothing 같은 효과를 내어 모델의 Robustness를 높여 일반화 성능을 높일 수 있다.

## Feature-based KD

- 내부 접근이 모두 가능할 때, 활용 가능한 기법으로 Teacher 모델의 Layer의 Feature를 지식으로 사용하는 기법이다.
- 즉 Layer의 크기가 다르더라도 중간에 나온 Feature의 결과는 비슷해지도록 하는 것이다.

![스크린샷 2024-12-27 123450](https://github.com/user-attachments/assets/825a154f-26dc-433b-9fd3-54b82318b484)

- Feature-based KD는 중간 Feature를 학습하는 것으로 중간 Feature에 대해서 MSE Loss를 사용한다. 또한 기본적으로 Cross-Entropy loss도 분류기에서 사용한다.
- MSE Loss를 계산 시에 주의할 점으로는 각 모델의 차원의 크기이다. Student 모델과 Teacher 모델은 차원이 다를 수 있기에 이에 따라 맞춰주는 작업이 필요하다.
- 이것을 맞춰주는 것이 regressor layer(R)로 smooth하게 차원을 맞춰준다. 이 regressor layer 또한 학습하는 것이다. 이를 수식으로 보면 아래와 같다.

![스크린샷 2024-12-27 124220](https://github.com/user-attachments/assets/c9b5229a-38e3-47c9-a508-8e6d83976cbf)

- 그렇다면 어떤 Layer에서 Feature를 추출해야 할까?
    - 일반적으로는 **중간 Layer**에서 추출한 Feature map을 사용한다.
    - 초기 Layer는 저수준의 Feature를 표현하여 특정 데이터에 덜 특화되어 있기에 distiilation의 효과가 적고, 후반 Layer는 지나치게 고수준의 정보만 담고 있어 Logit-based KD와 큰 차이를 보지 못한다.

## Imitation Learning

- **Black-box KD를 하는 방법**으로 다른 에이전트의 행동을 관찰하고 이를 모방하여 자신의 정책을 학습하는 기계 학습 방법이다.
- **Imitation Learning 과정**
    1. **모방 데이터 수집**
        - 공개된 데이터를 활용하거나, Teacher 모델에게 부탁하여 질문을 설정한다.
        - 이후 Prompt Engineering을 활용하여 다양한 답변과 구체적인 과정 및 설명을 답변하도록 한다. 이를 통해 Teacher모델은 더 유의미한 답변을 추출하게 된다.
    2. **데이터 전처리**
        - 수집된 데이터의 품질 확인 및 검증을 한다.
    3. **모델 학습**
        - 품질 검증이 끝난 데이터를 활용하여 모델 재학습을 한다.
- **Imitation Learning 데이터의 특징**
    - 모델의 출력으로 구성된 **인조 데이터**로서, 수집 비용이 저렴하지만, Teacher 모델의 오류나 비정확한 예측, 편향된 데이터 등이 포함되는 등 **Noisy한 데이터**이다.
    - 또한 이런 데이터는 **설명 가능한 답변**이라는 특징을 가진다. 즉 인간이 해석이 가능하다는 것이다.
- **Imitation Learning 장/단점**
    - 장점
        - 현재 Black-box모델의 유일한 KD 접근법이다.
        - 다른 KD 기법과 다르게 인간이 해석 가능한 형태의 지식을 가진다.
    - 단점
        - 출력된 응답만으로는 LLM의 내부 지식의 이해 및 심층적 학습이 어렵다.
        - LLM의 응답에 수렴하여 답변의 다양성이 떨어지거나 새로운 상황에 대한 대응력이 떨어질 수 있다. 즉 **출력의 다양성과 품질**이 학습 효과에 큰 영향을 끼친다.

## 추가적인 KD 방법론

- **Multi-teacher**
    - 여러 개의 Teacher로부터 평균적으로 학습하는 방법으로 일종의 앙상블 방법이다.
- **Cross-model**
    - 다른 Modality를 지닌 Teacher 모델에게 지식을 받는 방법론
