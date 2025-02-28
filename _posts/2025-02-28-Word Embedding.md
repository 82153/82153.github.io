---
layout: article
title: "[NLP] Word Embedding"
date: 2025-02-28 18:12 +0900
category: NLP
tags: [NLP]
aside:
  toc: true
sidebar:
  nav: NLP
---
## Vector Representation

1. **One-Hot Encoding**
    - 사전 안의 단어를 Categorical 변수로 Encoding한 벡터로 표현한 것
    - 사전의 길이만큼 vector의 크기를 만들고 **각 단어가 해당하는 vector의 Dimension의 값만 1**로 하고 **나머지 Dimension에 대해서는 0**으로 표현하는 방식이다.
    - 이 One-Hot Encoding의 결과(One-Hot vector)에서 각 vector의 내적은 항상 0이고, 유클리드 거리는 $\sqrt2$이다.
    - One-Hot vector는 대부분의 값이 0인데 이 값들을 다 저장하고 있어야 하기에 메모리적으로 비효율적이다. 그렇기에 **1의 값을 가지는 Dimension의 index만 저장하는 Spase Representation**으로 저장한다.
    - 근데 유사도와 거리가 항상 동일하면, 단어 간의 관계? 같은 것을 제대로 표현할 수 없지 않나 라는 생각이 든다.
2. **Distributed Vector Representation**
    - 단어의 의미를 여러 차원에 Non-Zero의 형태로 표현한 것
    - 위에서 내가 말한 단어 간의 관계(의미론적 유사성)을 나타내지 못하는 문제를 해결 할 수 있다.
    - 이런 방법을 **Word Embedding**이라 한다. 이 Word Embedding의 대표적인 방법론이 **주변 단어의 정보들을 이용해 word vector를 표현하는 방식이 Word2Vec**이다.

---

## Word2Vec

- Word2Vec의 핵심 아이디어는 특정 단어가 주어졌을 때, 그 **단어 주변의 다른 단어의 확률 분포를 모델링** 하는 것이다. 즉 이 확률 분포가 특정 단어의 의미를 결정한다는 것이다.
- Word2Vec에서는 각 단어의 Dense Vector를 얻기 위해 2가지 예측 task를 수행하는데, **주변 단어를 통해 가운데의 단어를 예측하는 Continuous Bag of Words**와 **가운데 주어진 단어를 통해 주변 단어를 예측하는 Skip-gram**이라는 방법을 사용했다.(주로 Skip-gram을 사용)

### Skip-gram

- 단어가 주어졌을 때, 앞 뒤로 몇 개의 단어를 예측할지 Window-size라는 변수를 통하여 설정한다.

![image.png](attachment:5f36120b-b475-41eb-9909-64e5588bf680:image.png)

- Word2Vec에서는 vocabulary를 만들고, 문장 내에 각 단어에 대해서 One-Hot Embedding을 진행한다. 이후 각 단어에 대해서 2개의 layer를 거친 후, Softmax를 거쳐, 주변에 올 단어의 확률 분포를 구한다.
- 이때 각 layer 사이에 비선형 함수를 적용하지 않는다. 이를 바탕으로 행렬 연산을 분석해보면, input word 벡터는 one-hot 벡터로 1로 지정된 Dimension과 동일한 $W_{1}$의 index의 column이 결과로 나올 것이고, 이후 $W_2$와 행렬 곱 연산을 진행한다.
- 위의 그림에서 보면 우리가 학습하는 방향은 $W_1$의 주어진 단어 ‘study’ 벡터와 $W_2$의 ‘math’ 벡터의 내적이 높아지는 방향으로 학습한다. 즉  **$W_1$의 중심 단어 벡터와 $W_2$의 GT 주변 단어 벡터의 내적이 커져야 한다.**