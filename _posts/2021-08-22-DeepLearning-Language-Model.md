---
title: "[딥러닝] 언어모델, RNN, GRU, LSTM, Attention, Transformer, GPT, BERT 개념 정리"
author: Seungjoo Ra
date: 2021-08-22 16:00:00 +0900
categories: [DeepLearning, Natural Language]
tags: [DeepLearning, Language Model]
math: true
mermaid: False
---

---
**Contents**

{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}
---

# 언어 모델(Language Model)

언어 모델이란 문장과 같은 단어 시퀀스에서 `각 단어의 확률을 계산하는 모델`이다.

# 통계적 언어 모델(Statistical Language Model, SLM)

통계적 언어 모델에서는 `단어의 등장 횟수를 바탕으로 조건부 확률을 계산`한다.<br>
통계적 언어모델은 횟수 기반으로 확률을 계산하기 때문에 `말뭉치에 등장하지 않는 새로운 단어를 생성하지 못하는 희소문제`를 가지고 있다.

# 신경망 언어 모델(Neural Langauge Model)

신경망 언어 모델에서는 횟수 기반 대신 `Word2Vec 이나 fastText 등의 출력 값인 임베딩 벡터를 사용`한다.<br>
그렇기 때문에 말뭉치에 등장하지 않더라도 의미적, 문법적으로 유사한 단어라면 선택될 수 있는 특징을 가지고 있다.

# 순환 신경망 (RNN, Recurrent Neural Network)

RNN은 `연속형 데이터를 잘 처리하기 위해 고안된 신경망`이다.

## RNN의 구조

![](blog_img/2021_08_22_DeepLearning_Language_Model/RNN.png){: .shadow}_RNN 구조_

은닉 벡터가 다음 시점의 입력 벡터로 다시 전달되는 특성 때문에 `순환(Recurrent) 신경망` 이라고 불린다.

## RNN의 단점

- <span style="color:#C0392B"><b>병렬화 불가능</b></span><br>

RNN의 구조는 벡터가 순차적으로 입력되어 연속적인 데이터를 처리할 수 있게 해주지만 이러한 특성으로 인해 `GPU연산의 장점인 병렬화가 불가능하다는 단점`이 있다.


![](blog_img/2021_08_22_DeepLearning_Language_Model/tanh.png){: .shadow}_tanh의 미분_

- <span style="color:#C0392B"><b>기울기 소실 및 폭발</b></span><br>

역전파 과정에서 활성화 함수 tanh의 미분(위의 그림)값을 반복해서 곱해주게 되는데 만약 이 값이 0.9 일 때 10제곱이 된다면 0.349가 된다.<br>
이렇게 되면 시퀀스 앞쪽에 있는 hidden-state 벡터에는 역전파 정보가 거의 전달되지 않게 되는 현상이 발생하게 되는데 이와 같은 문제를 `기울기 소실(Vanishing Gradient)`이라고 한다.<br>

반대로 이 값이 1.1 이면 10제곱만해도 2.59배로 커지게 된다.<br>
이렇게 되면 시퀀스 앞쪽에 있는 hidden-state 벡터에는 역전파 정보가 과하게 전달되는 현상이 발생하게 되는데 이와 같은 문제를 `기울기 폭발(Exploding Gradient)`이라고 한다.<br>

# 장단기 기억망(Long Term Short Memory, LSTM)

RNN의 단점인 `기울기 소실 문제를 해결하기 위해 gate를 추가한 모델`이다.

## LSTM의 구조

![](blog_img/2021_08_22_DeepLearning_Language_Model/LSTM.png){: .shadow}_LSTM 구조_

- **gate의 역할**

    * forget gate : 과거 정보를 얼마나 유지할 것인가?
    * input gate : 새로 입력된 정보는 얼만큼 활용할 것인가?
    * output gate : 두 정보를 계산하여 나온 출력 정보를 얼마만큼 넘겨줄 것인가?

활성화 함수를 거치지 않는 상태인 cell state($C_t$)가 추가되었는데 cell-state는 역전파 과정에서 활성화 함수를 거치지 않아 정보 손실이 없기 때문에 뒷쪽 시퀀스의 정보에 비중을 결정할 수 있으면서 동시에 앞쪽 시퀀스의 정보를 완전히 잃지 않을 수 있게 된다.

# GRU(Gated Recurrent Unit)

![](blog_img/2021_08_22_DeepLearning_Language_Model/GRU.png){: .shadow}_GRU 구조_

GRU는 LSTM 간소화된 버전이라고 할 수 있다.<br>

- ## GRU의 특징
1. LSTM에서 있었던 cell-state가 사라지고 $C_t$와 $h_t$가 하나의 벡터 $h_t$로 통합되었다.
2. 하나의 $Z_t$ gate가 forget, input gate 모두를 제어하는데 $Z_t$가 1일 경우 forget gate만 열리고, 0일 경우 input gate만 열리는 것과 같은 효과를 나타내게 된다.
3. GRU 셀에서 output gate가 사라진 대신에 전체 상태 벡터 $h_t$ 가 각 time-step에서 출력되며, 이전 상태의 $h_{t-1}$ 의 어느 부분이 출력될 지 새롭게 제어하는 Gate인 $r_t$ 가 추가되었다.

- ## 장기 의존성 문제

RNN이 가지는 가장 큰 단점은 기울기 소실로부터 나타나는 `장기 의존성(Long-term dependency)문제`인데 이 문제는 문장이 길어질 경우 앞 단어의 정보를 잃어버리게 된다.<br>
이와 같은 장기 의존성 문제를 해결하고자 나온 것이 셀 구조를 개선한 LSTM, GRU 이다.

# Attention

RNN기반 모델인 LSTM과 GRU는 구조적으로 고정 길이의 hidden state 벡터에 모든 단어의 의미를 담아야 하기 때문에 문장이 길어지면 모든 단어 정보를 고정 길이 벡터에 담기 어렵다는 문제가 있다.<br>
이와 같은 문제를 해결하기 위해 고안된 것이 **Attention** 이다.

## Attention의 동작

- ### 인코더 동작

1. 각 인코더의 Time-step마다 생성되는 hidden state 벡터를 간직한다.
(입력단어가 n개라면 n개의 벡터를 모두 간직하게 된다.)
2. 모든 단어가 입력되면 생성된 hidden state 벡터를 디코더에 넘겨준다.

- ### 디코더

**결제시스템 아이디어**

![](blog_img/2021_08_22_DeepLearning_Language_Model/Decoder_idea.png){: .shadow}

1. 찾고자 하는 정보에 대한 검색어(Query)를 입력
2. 검색 엔진은 검색어와 가장 비슷한 키워드(Key)를 검색
3. 키워드(Key)와 연결된 페이지(Value)를 출력

- 디코더의 검색엔진 활용동작
1. 디코더의 각 time-step 마다의 hidden-state 벡터는 쿼리(query)로 작용한다.
2. 인코더에서 넘어온 N개의 hidden-state 벡터를 키(key)로 여기고 이들과의 연관성을 계산한다.
3. 이 때 계산은 내적(dot-product)을 사용하고 내적의 결과를 Attention 가중치로 사용한다.

- #### 디코더 동작

![](blog_img/2021_08_22_DeepLearning_Language_Model/Decoder.png){: .shadow}

1. 쿼리(Query)인 디코더의 hidden state 벡터, 키(Key)인 인코더에서 넘어온 hidden state 벡터를 준비한다.

2. 각각의 벡터를 내적한 값을 구한다.

3. 소프트 맥스 함수를 취해주어 어떤 단어가 연관성이 높은지 계산한다.

4. 소프트 맥스를 통해 구해진 값에 밸류(Value)에 해당하는 인코더에서 넘어온 hidden state 벡터를 곱해준다.

5. 위의 연산을 통해 추출된 벡터들을 모두 더해주게 되면 결과적으로 벡터의 성분 중 쿼리(Query)와 키(Key) 연관성이 높은 밸류 벡터의 성분이 더 많이 들어있게 된다.

6. 최종적으로 디코더의 hidden state 벡터를 사용하여 출력 단어가 결정된다.

- ## Attention 총 정리
디코더는 인코더에 넘어온 모든 hidden state 벡터에 대해 위의 계산을 반복한다.<br>
그렇기 때문에 time-step 마다 출력할 단어가 어떤 인코더의 어떤 정보와 연관되어 있는지, 즉 어떤 단어에 집중(Attention)할 지 알 수 있게 된다.<br>
`Attention을 활용하게 되면 디코더가 인코더에 입력되는 모든 단어의 정보를 활용할 수 있기 때문에 결과적으로 장기 의존성 문제를 해결할 수 있다.`

# Transformer

모든 토큰을 동시에 받아 연산하기 때문에 `병렬연산이 가능`한다.<br>
인코더 블럭과 디코더 블럭이 각각 6개씩 모여있는 구조이다.<br>
인코더 블록과 디코더 블록은 각각 2개 3개로 나눌 수 있는데 그 구조는 다음과 같다.<br>
- 인코더 블록
1. 
sub-layer(Multi-Head(self) Attention, Feed Forward)로 나눌 수 있고, 디코더 블록은 3개의 sub-layer(Masked Multi-Head(self) Attention, Multi-Head (Encoder-Decoder) Attention, Feed Forward)로 나눌 수 있다.<br>
기존의 Attention과는 다르게 각 벡터가 모두 가중치 벡터라는 특징을 가지고 있다.

## Self Attention

트랜스포머에서 번역하려는 문장 내부 요소의 관계를 잘 파악하기 위해서 문장 자신에 대해 어텐션 매커니즘을 적용하게 되는데 이것이 Self Attention 이다.<br><br>
**쿼리(Query)**: 분석하고 하는 단어에 대한 가중치 벡터<br>
**키(Key)** : 각 단어가 쿼리에 해당하는 단어와 얼마나 연관있는 지를 비교하기 위한 가중치 벡터<br>
**밸류(Value)** : 각 단어의 의미를 살려주기 위한 가중치 벡터<br>

- ### self Attention 과정
1. 특정 단어의 쿼리(q) 벡터와 모든 단어의 키(k) 벡터를 내적한다.. 내적해서 나온느 값은 Attention Score가 된다.
2. 트랜스포머에서는 이 가중치를 q(query),k(key),v(value) 벡터 차원 $d_k$ 의 제곱근인 $\sqrt d_k$로 나누어 준다.(계산값을 안정적으로 만들어주기 위한 계산보정)
3. Softmax를 취해주어 쿼리에 해당하는 단어와 문장 내 다른 단어가 가지는 관계의 비율을 구할 수 있다.
4. v(value)인 각 단어의 벡터를 곱해준 후 모두 더한다.

## Transformer Encoding 

- ### Multi-Head Attention
Self-Attention을 동시에 여러개로 수행하는 것이다.<br>
각 Head 마다 다른 Attention 결과를 내어주기 때문에 앙상블과 유사한 효과를 얻을 수 있다.

- ### Layer Normalization, Skip connection
Layer normalization의 효과는 Batch normalization과 유사하다.<br>
학습이 훨씬 빠르고 잘 되도록 하는 효과가 있다.<br>
Skip connection(혹은 Residual connection)은 역전파 과정에서 정보손실을 막는 역할을 한다.

- ### Feed Forward Neural Network
은닉층의 차원이 늘어났다가 다시 원래 차원으로 줄어드는 단순한 2층 신경망이다.
활성화 함수로 Relu를 사용한다.

## Transformer Decoding

- ### Masked Self-Attention
디코더 블럭에서 사용되는 특수한 Self-Attention 이다.<br>
디코더는 Auto Regressive하게 단어를 생성하기 때문에 타깃 단어 이후 단어를 보지 않고 단어를 예측해야 한다.<br>
따라서 타깃 단어 뒤에 위치한 단어는 Self-Attention에 영향을 주지 않도록 마스킹(Masking)을 해주어야 한다.<br>
그렇기 때문에 Softmax를 취해주기 전에 가려주고자 하는 요소에만 $-\infty$에 해당하는 매우 작은 수를 더해준다.

- ### Encoder-Decoder Attention
디코더에서 Masked Self-Attention 층을 지난 벡터는 Encoder-Decoder Attention 층으로 들어가게 되는데 `좋은 번역을 위해서 번역할 문장과 번역되는 문장의 정보를 엮어주는 역할을 하는 것이 Encoder-Decoder Attention` 이다.<br>
디코더 블록의 Masked Self-Attention으로부터 출력된 벡터를 쿼리(q) 벡터로 사용한다.<br>
키(k)와 밸류(v) 벡터는 최상위(6번째)인코더 블록에서 사용했던 값을 그대로 가져와서 사용하게 된다.<br>
계산 과정은 Self-Attention과 동일하다.

- ### Linear & Softmax Layer
디코더의 최상층을 통과한 벡터들은 Linear층을 지난 후 Softmax를 통해 예측할 단어의 확률을 구하게 된다.

- ### Positional Encoding
Transformer의 병렬처리로 인해 각 단어들의 위치를 컴퓨터에게 전달해야 하기 때문에 `단어의 상대적인 위치 정보를 제공하기 위한 벡터를 만드는 과정을 Positional Encoding`이라고 한다.


# 사전 학습 모델(Pre-trained Language Model)
대량의 데이터를 사용하여 미리 학습하고 학습된 모델에 추가로 필요한 데이터를 학습시켜 최종적으로 모델의 성능을 최적화 한다.<br>
전이 학습(Transfer Learning)이라고도 한다.<br>
사전학습 모델은 `사전학습(Pre-training)과 사전학습이 끝난 모델에 하고자하는 테스크에 특화된 데이터를 학습시키게 되는데 이것을 Fine-tuning`이라고 한다.<br>
Fine-tuning에서는 학습시 레이블링 된 데이터(감성 분석, 자연어 추론, 질의 응답)를 사용하게 되고 최종적으로 이 두가지 과정을 통해 모델이 완성되게 된다.<br>
`GPT, BERT`가 여기에 해당한다.

## GPT(Generative Pre-trained Transformer)
Transformer의 디코더 블럭을 12개 쌓아올려 만든 모델

## BERT(Bidirectional Encoder Represntation by Transformer)
Transformer의 인코더 블럭만 12개 쌓아올려 만든 모델<br>
[CLS],[SEP]와 같은 special token을 가지고 있는 특징이 있다.<br>
**CLS(Classification)** : 입력의 맨 앞에 위치하는 토큰<br>
**SEP(Separation)** : 사전학습시 텍스트를 두 부분으로 나누어 넣게 되는데 첫번째 문장의 끝과 두번째 문장의 끝에 위치하여 첫번째 문장과 두번째 문장을 나누어주는 역할<br>

- ### BERT Input Vector

- **Token Embeddings** : 단어를 나타내는 임베딩, Word2Vec, GloVe, FastText 등으로 사전 학습된 임베딩 벡터를 사용

- **Segment Embeddings** : 첫 번째 부분과 두 번째 부분을 구분하기 위한 임베딩
[CLS]부터 첫 번째 [SEP] 까지 동일한 벡터를 적용하고, 다음 토큰부터 두 번째 [SEP] 까지 동일한 벡터를 적용

- **Positional Embeddings** : 단어의 위치를 나타내기 위한 임베딩

### BERT의 사전학습 방법

- MLM(Masked Language Model)
BERT는 빈칸 채우기를 하면서 언어를 학습하게 되는데 사전 학습 과정에서 레이블링이 되지 않은 말뭉치 중에서 랜덤으로 15%가량의 단어를 마스킹한다.<br>
그리고 마스킹된 위치의 단어를 예측하는 방식으로 학습을 진행한다.<br>
`GPT와 BERT의 차이점은 GPT는 순방향으로 예측하여 단어를 생성하고, BERT는 양방향으로 학습하여 빈칸의 단어를 예측하기 때문에 문맥 사이에 가진 의미를 최대한 학습할 수 있다.`

- NSP(Next Sentence Prediction)
모델이 문맥에 맞는 이야기를 하는지 아니면 동문서답을 하는지를 판단하며 학습하는 방식이다.


# 마무리
공부하면서 간단하게 나마 이해를 하기 위해 심도있게 들어가지 않고 기본적인 개념정도만 짚고 가자는 의미에서 정리해보았습니다.<br>
언어모델 쪽으로 더 공부하게 되면 추가적으로 업데이트 하겠습니다.<br>
잘못된 부분이나 추가로 알려주고 싶으신 부분이 있으시면 댓글로 작성해주시면 감사하겠습니다!<br>




