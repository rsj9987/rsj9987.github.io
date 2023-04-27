---
title: "자연어(Natural Language)"
author: Seungjoo Ra
date: 2021-08-17 18:00:00 +0900
categories: [Natural Language]
tags: [Natural Language, NLP]
math: true
mermaid: False
---

---
**Contents**

{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}
---


# 자연어(Natural Language)

자연어란 사람들이 일상적으로 쓰는 언어를 인공적으로 만들어진 언어인 인공어와 구분하여 부르는 개념이다.

자연어가 아닌 것으로는 `에스페란토어, 코딩 언어 등`이 있다.

이러한 자연어를 컴퓨터로 처리하는 기술을 자연어 처리(Natural Language Processing, NLP)라고 한다.

# 자연어 처리로 할 수 있는 것

## 1. 자연어 이해(NLU, Natural Language Understanding)
- 분류
뉴스기사 분류, 감성 분석(Positive, Negative)

- 자연어 추론(NLI, Natural Language Inference)
전제 : A는 B를 살해했다. 가설 : B는 살인자다. -> 사실 혹은 거짓
- 기계 독해 (MRC, Machine Reading Comprehension), 질의 응답(Q&A, Question & Answering)
비문학 문제 풀기
- 품사 태깅(POST tagging), 개체명 인식(Named Recognition) 등

## 2. 자연어 생성(NLG Natural Language Generation)
- 텍스트 생성
예) 뉴스기사 생성, 가사 생성

## 3. 자연어 이해와 생성의 복합 사용

- 기계 번역(Machine Translation)
- 요약(Summarization)
문서내에서 해당 문서를 가장 잘 요약하는 부분을 찾아내는 추출 요약(NLU에 가까움)
문서를 요약하는 요약문을 생성하는 생성 요약(NLG에 가까움)
- 챗봇(Chatbot)
식당 예약 챗봇, 상담 응대 챗봇 등


이외에도 TTS(Text to speech), STT(Speech to text), Image Captioning 등 음성, 이미지와 언어가 결합된 기술도 있다.

# 벡터화(Vectorize)

컴퓨터는 자연어 자체를 받아들일 수 없기 때문에 컴퓨터가 이해할 수 있도록 자연어를 숫자형 벡터로 바꾸어 주어야 하는데 이것을 벡터화라고 한다.
자연어를 벡터화 하는 방법은 크게 2가지로 나눌 수 있다.

## 1. 등장 횟수 기반 단어 표현(Count-based Representation)
단어 혹은 문서, 문장에 등장하는 횟수를 기반으로 벡터화 하는 방법

- Bag-of-Words(CounterVectorizer)
- TF-IDF(TfidfVectorizer)

## 2. 분포 기반의 단어 표현(Distributed Representation)
타겟 단어 주변에 있는 단어를 기반으로 벡터화 하는 방법
- Word2Vec
- Glove
- fastext

# 텍스트 전처리(Text Preprocessing)

## 정규표현식(Regax)

정규표현식이란 문자열을 다루기 위한 강력한 패키지이다. 벡터화 하기 전 전처리 과정에서 사용될 수 있다.

[점프 투 파이썬 정규표현식](https://wikidocs.net/4308)에 잘 나와있으니 참고하여 실습해보자.


## 불용어(Stop words) 처리

분석에 큰 의미가 없는 단어를 지칭한다. 예를 들어 the, a, is, I 와 같은 문장을 구성하는 필수요소이지만 문맥적으로 큰 의미가 없는 단어가 불용어에 속한다.<br>
이러한 단어는 글이나 문장에 자주 나타나기 때문에 컴퓨터는 중요한 단어라고 인식할 수 있다. 하지만 특정 의미를 나타내주지 않는 단어이기 때문에 사전에 제거를 해줘야 한다.

**Spacy를 사용한 예제**

```python
import spacy
from spacy.tokenizer import Tokenizer

nlp = spacy.load("en_core_web_sm")

list(nlp.Defaults.stop_words)[:30]
```
```
['or','while', 'also', '‘m', 'although', 'many',
'eleven','thereupon', 'these', 'third', 'ten', '‘ll', 'against',
'enough', 'yourselves', 'by', 'whom', 'but', '’m', 'unless', 
'therein', 'neither', 'throughout', 'nothing', 'was', 'herself',
'nine', 'seem', 'how', 'between']
```

```python
example = "Family is not an important thing. It's everything."

tokenizer = Tokenizer(nlp.vocab)

tokens = tokenizer(example)
result = []
for token in tokens:
    if token.is_stop == False:
        result.append(token)
print(result)
```

```
[Family, important, thing., It's, everything.]
```

**불용어 커스터마이징**

```python
# stop words 추가
STOP_WORDS = nlp.Defaults.stop_words.union(["It's"])

result = []
for token in tokens:
    if token.text not in STOP_WORDS:
        result.append(token)
print(result)
```
```
[Family, important, thing., everything.]
```

## 어간 추출(Stemming)

단어의 의미가 포함된 부분으로 접사 등이 제거된 형태이다.<br>
어근이나 단어의 원형과 같지 않을 수 있다는 단점이 있다.<br>
예를 들어 excite, exciting, excited, excites의 어간은 단어들의 뒷부분이 제거된 excit가 된다.<br>
어간 추출은 ing, ed, s와 같은 단어를 제거하게 된다.<br>
Stemming은 알고리즘이 간단하여 속도가 빠르기 때문에 속도가 중요한 검색 분야에서 많이 사용되고 있다.<br>

```python
from nltk.stem import PorterStemmer

ps = PorterStemmer()

words = ['excite', 'exciting','excited', 'excites']

for word in words:
    print(ps.stem(word))
```
```
excit
excit
excit
excit
```

## 표제어 추출(Lemmatization)

표제어 추출은 어간추출보다 체계적이다.<br>
단어들의 기본 산전형 단어 형태인 표제어(Lemma)로 변환한다.<br>
명사의 복수형은 단수형으로, 동사는 모두 타동사로 변환된다.<br>

```python
lem = "The social wolf. Wolves are complex."

nlp = spacy.load("en_core_web_sm")

doc = nlp(lem)

for token in doc:
    print(token.text," || ", token.lemma_)
```
```
The  ||  the
social  ||  social
wolf  ||  wolf
.  ||  .
Wolves  ||  wolf
are  ||  be
complex  ||  complex
.  ||  .
```




