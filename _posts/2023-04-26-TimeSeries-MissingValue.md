---
title: "시계열 데이터의 결측치 처리 방법"
author: Seungjoo Ra
date: 2023-04-26 18:00:00 +0900
categories: [TimeSeries]
tags: [Preprocessing]
math: true
mermaid: False
---

---
**Contents**

{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}
---






# 시계열 데이터의 결측치 처리 방법

## 1. 누락된 데이터의 대표적인 해결 방법

- 대치법(Imputation)
  데이터셋 전체의 관측에 기반하여 데이터를 채워 넣는 방법

- 보간법(Interpolation)
  대치법의 한 형태로 인접한 데이터를 사용하여 누락된 데이터를 추정하는 방법

- 결측치 제거
  누락된 데이터의 기간을 완전히 사용하지 않는 방법
* 알고 가야할 **'사전관찰'**
  사전관찰이란 미래의 어떤 사실을 안다는 뜻으로 사용된다. 즉 데이터를 통해 실제 알아야 하는 시점보다 더 일찍 미래에 대한 사실을 발견하는 방법이다.
  사전관찰은 어떻게든 미래에 일어날 일에 대한 정보가 모델에서 시간을 거슬러 전파되어 모델의 초기 동작에 영향을 주는 방법이다.

## 2. 포워드 필(Forward Fill)

포워드 필은 누락된 값이 나타나기 직전의 값으로 누락된 값을 채우는 가장 간단한 방법

python은 판다스의 dataframe.ffill() 함수를 사용할 수 있다.

원본 데이터와 랜덤하게 누락된 데이터를 비교해보면 누락된 부분은 누락되기 전 값으로 대치된 것을 확인할 수 있다.

```python
rand_df["ffill"] = rand_df["Unrate"].ffill()
plt.plot("Date", "Unrate", "ro--", data=df.iloc[350:400], label="origin");
plt.plot("Date", "ffill", "bo--", data=rand_df.iloc[350:400], label="random ffill");
plt.legend()
plt.show()
```

![](blog_img/2023-04-26-TimeSeries-MissingValue/ffill.png)

## 3. 이동평균(Moving Average)

관측 값들의 집합에서 특정 크기의 부분 집합만큼 연속적으로 이동하며 산출한 부분 평균

과거의 값으로 미래의 값을 예측한다는 관점에서 보면 포워드 필과 유사하지만 이동평균은 최근 과거의 여러 시간대를 입력한 내용을 사용한다는 점에서 다르다.

전체 평균에 관한 개별 데이터 값을 '의심'할 만한 이유가 있다면 포워드 필 보다는 이동평균을 사용해야 한다. 포워드 필에 의해 채워진 값은 누락된 값의 '실제' 관측 값보다 임의의 노이즈를 포함한다. 반면에 평균은 노이즈의 일부를 제거할 수 있다.

> 이동평균이 반드시 산술평균일 필요는 없다.
> 
> <u>지수가중이동평균</u>을 사용하여 비교적 최근 데이터에 더 많은 가중치를 줄 수도 있고 <u>기하평균</u>을 사용하여 강한 상관관계를 가지며 시간이 지나면서 복합적인 값을 가지는 시계열 데이터에 유용하여 사용할 수 있다.

- ### 사전관찰이 없는 이동 평균

```python
rand_df["MA_NoLookAhead"] = rand_df["Unrate"].rolling(window=3, min_periods=1).mean()
plt.plot("Date", "Unrate", "ro-", data=df.iloc[350:400], label="origin")
plt.plot("Date", "MA_NoLookAhead", "b--", data=rand_df.iloc[350:400], label="NoLookAhead")
plt.legend()
plt.show()
```

![](blog_img/2023-04-26-TimeSeries-MissingValue/ma_nolookahead.png)

- ### 사전 관찰이 있는 이동 평균

```python
rand_df["MA_LookAhead"] = rand_df["Unrate"].rolling(window=3, min_periods=1, center=True).mean()
plt.plot("Date", "Unrate", "ro-", data=df.iloc[350:400], label="origin")
plt.plot("Date", "MA_LookAhead", "b--", data=rand_df.iloc[350:400], label="LookAhead")
plt.legend()
plt.show()
```

![](blog_img/2023-04-26-TimeSeries-MissingValue/ma_lookahead.png)

- ### 이동 평균을 사용하기 전에 고려해봐야할 것
1. 이동평균으로 결측치를 대체할 때는 미래 예측을 위한 과거의 데이터만으로 이동평균 값을 구할 수 있는지 사전관찰을 수월하게 만들 수 있을지 고려해봐야 한다.

2. 사전관찰을 고려하지 않는 상황이라면 최상의 추정치에는 누락된 데이터의 전과 후가 포함되므로 추정에 사용되는 정보를 최대화할 수 있다. 

3. 과거와 미래에 대한 정보를 사용하는 것은 시각화와 기록 보관용 애플리케이션에는 유용하지만 예측 모델의 입력 데이터를 준비할 때는 적합하지 않다.

4. 이동평균에 의한 데이터 대치의 경우 데이터 셋의 분산을 줄여준다. 이는 오류 지표의 계산 시 시계열 모델의 성능을 '과대 평가'할 수 있다는 것을 기억해야 한다.

## 4. 보간법(Interpolation)

전체 데이터를 기하학적인 행동에 제한하여 결측치를 결정하는 방법

대표적인 방법으로 선형 보간법(Linear Interpolation)이 있다. 선형보간법은 결측치가 주변 데이터에 선형적인 일관성을 갖도록 제한하는 방법이다.

이동평균과 마찬가지로 과거 미래 데이터의 활용 여부를 결정하는 것이 중요하다.

- ### 선형 보간법과 다항식 보간법

```python
inter_df = pd.DataFrame(rand_df["Date"])
inter_df["LinearInterpolation"] = rand_df["Unrate"].interpolate(method="linear")
inter_df["PolynomialInterpolation"] = rand_df["Unrate"].interpolate(method="polynomial", order=2)
plt.plot("Date", "Unrate", "ro--", data=df[350:400], label="origin")
plt.plot("Date", "LinearInterpolation", "b-", data=inter_df[350:400], label="linear")
plt.plot("Date", "PolynomialInterpolation", "g-", data=inter_df[350:400], label="Polynomial")
plt.legend()
plt.show()
```

![](blog_img/2023-04-26-TimeSeries-MissingValue/Linear_Polynomial.png)

선형적으로 증가하는 추세를 가진 데이터셋에 결측치를 대체할 때 이동평균을 사용한다면 실제 추세 대비 누락된 값이 일정하게 과소 평가될 수 있다. 그러므로 이런 경우 선형 보간법을 적용하는 것이 적절할 수 있다.

### 5. 전체 결측치 대체 방법 비교

MSE(Mean Square Error)를 통해서 비교

```python
err_df = pd.DataFrame()
for col in rand_df.columns[2:]: # Date와 Unrate 제외
    err_df[col] = (df["Unrate"] - rand_df[col])**2

err_df.mean().sort_values()
```

```
PolynomialInterpolation    0.045087
LinearInterpolation        0.048334
MA_LookAhead               0.062757
ffill                      0.159989
MA_NoLookAhead             0.206944
dtype: float64
```

#### ! 주의 사항

> pandas.DataFrame.interpolate 함수는 데이터 타입이 'object'일 경우에 보간을 적용해주지 않고 원본 데이터를 그대로 return 해준다. 만약 interpolate가 정상적으로 되지 않을 경우 데이터 타입이 수치형인지 확인해야 한다.
