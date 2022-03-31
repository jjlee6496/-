# 3장  Performance Evaluation
- 과적합: 예측모형의 noise까지 fitting하는 상황. ex)한사람과 오래 사귀면 다른사람과 새로 시작하기 어렵다... 그사람에게 맞춘것이 많기 때문에...
  - 모형적합과 성과평가를 위해 동일한 데이터를 이용할 때 발생하는 일종의 bias

- 과적합된 모형은 새로운 데이터해 대해 낮은 성과평과를 보일 가능성 높음. 따라서 모형적합, 성과평가시 전체 data를 분할하여 서로다른 data 사용이 바람직함.

#### Partitioning Cross-Sectional Data
일반적인 partitioning
- full data = training partition + validation partition + test partition
- training(model fitting에 사용), validation(assesment에 사용), test(무작위로 분할)

시계열 데이터의 분할(Temporal Partitioning)
- Full period = training period + validation period
- test period는 없음. 최신데이터를 모델에 사용하는게 타당
- test 무작위분할 x

#### Joining Partitions for Forecasting

- train, validation을 통해 선택한 최종 모형에 '전체 데이터'를 적용하여 미래값을 예측 => 예측을 위해 최신데이터를 사용하겠다는 취지.
- 또는, 전체 데이터를 이용하여 최종 모형을 다시 추정한 후 예측에 사용

평가용 기간의 선택
- 제1의 원칙: 예측기간과 동일한 길이 선택.
- 정리: 1. 분할 2. 학습 3. 평가 4.예측 의 순서로 분석

## 3.2 Naive Forecasts
- The k-step ahead naive forecast at time t: $`F_{t+k} = y_t`$
- The k-step ahead naive forecast at time t for seasonal series with M seasons: $`F_{t+k} = y_{t-M+k}`$
- simplest model이자 baseline model

## 3.3 Measuring Predictive Accuracy
적합도(Goodness of Fit) vs. 예측정확도(Predictive Accuracy or Forecasting Accuracy)  
- 적합도: 인과관계의 강도. 예) $R^2$ => training period에 대해 측정  
- 예측 정확도: 모형 적합에 사용되지 않는 데이터에 대한 예측력 => validation period에 대해 측정  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8251;Validation period에 결측치가 있는 경우는 '반드시 제외'하고 측정  
#### 자주 사용되는 예측 정확도의 측도
- t시점 개별 데이터에 대한 예측오차:$`e_{t} = y_{t}-F_{t}`$
- training period: $`t=1,2,...,n`$
- validation period: $`t=n+1,n+2,...,n+v`$

- $`MAE = \frac{1}{v}\sum_{t=n+1}^{n+v}|e_t|`$ :절대오차의 평균
- $`AE = \frac{1}{v}\sum_{t=n+1}^{n+v}e_t`$ :평균적으로 과대 or 과소
- $`MAPE = \frac{1}{v}\sum_{t=n+1}^{n+v}|\frac{e_t}{y_t}|\times100`$
  - 측정단위가 서로 다른 시계열의 예측성과 비교시 유용(상대오차, scale-free)
  - 예측오차의 방향성($e_t>0,e_t<0$)의 영향이 비대칭적임
$`RMSE = \sqrt{\sum_{t=n+1}^{n+v}({e_t})^2}=\sqrt{MSE}`$
$`RPMSE = \sqrt{\sum_{t=n+1}^{n+v}(\frac{e_t}{y_t})^2\times100}`$ :MAPE와 같은 취지  

:notebook_with_decorative_cover: 위 측도들을 training period에 사용하면 적합도, valid에 사용하면 예측정확도.  

#### 0값 계산
- $`y_t`$둘중 0이 있는 경우 MAPE, RPMSE등의 상대오차측도의 분모가 0이 되는 문제 발생.
- 대안  

```Math
$$MASE(Mean Abolute Scaled Error)=\frac{Validation MAE}{training MAE of naive forecasts}=\frac{\frac{1}{v}\sum_{t=n+1}^{n+v}|e_t|}{\frac{1}{n-1}\sum_{t=2}^{n}|y_{t-1}-y_t|}$$
```
  
:notebook_with_decorative_cover: MASE의 의미  
- 분모계산시 training period 이용
- 분모가 평소 변동폭 의미
- 추세와 계절성이 있는 시계열에 적합.(분모가1시점씩이동하고, 분모 분자가둘다 error이기 때문에)

#### Forecast Accuracy vs. Profitability
- 예)투자전략을 개발하기 위한 주가 예측 시계열 모델링 => 작은 예측오차가 투자전략의 성공 보장 X. 투자성과 기준으로 모형을 선택하는 것이 타당하다.

## 3.4 Evaluating Forecast Uncertainty
#### 예측 오차의 분포
- 평균적인 예측오차가 작아도 일부 극단적인 크기의 예측오차가 존재하는 경우 문제가 될 수 있음.
- 예측오차 내에 모델링 가능한 체계적인 패턴이 남아있을 수 있음.
- Error 확률분포에 대한 추가적인 모델링 필요할 수 있음.
∴ 잔차의 시도표, 히스토그램을 그려볼 필요 있음(잔차분석 필요!)  
#### 예측구간
- 예측의 불확실성을 반영하여 구한 예측값의 범위  
![잔차예측구간.PNG](https://github.com/jjlee6496/one-by-one/blob/main/%EC%8B%9C%EA%B3%84%EC%97%B4%EB%B6%84%EC%84%9D/images/PredictionInterval.PNG?raw=true)

- 방법1: 예측오차$`e_{t} = y_{t}-F_{t}`$ 가 정규분포를 따를 때. 예측치$`F_t`$에 $`SE[e_t]`$의 상수배를 가감하여 예측구간을 구함. But, 예측오차 $`e_t`$가 항상 정규분포를 따르진 않는다.
- 방법2: training period에서 계산된 $`	\{e_t|t\in train \}`$ 의 히스토그램을 $`e_{t+1}`$의 확률분포로 간주, 이 분포의 percentile을 $`F_{t+1}`$에 가감하여 예측구간을 구함.
:notebook_with_decorative_cover: 1.$`y_{t}$와$F_{t}`$ 2.$`\hat{e_{t}}(=y_{t}-F_{t})`$의 시도표 3.$`\{e_t|t\in train \}`$의 히스토그램 4.$`y_{t}$와$F_{t}`$의 prediction interval 시도표  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8251;4번에서 먼 미래의 예측범위와 가까운 미래의 예측범위가 같은게 맞을까?  

#### 예측원뿔(Prediction Cone)
- 상식: 가까운 미래에 대한 예측의 불확실성 $`\leq`$ 먼 미래에 대한 예측의 불확실성
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ∴ 가까운 미래에 대한 예측구간의 길이 $`\leq`$ 먼미래에 대한 예측구간의 길이 => "예측원뿔"  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp&nbsp;&nbsp;&nbsp;모델링을 통해 이러한 결과가 나오도록 할 수 있음. 대부분의 예측 모형은 이 상식을 구현함.  
![예측원뿔.PNG](https://github.com/jjlee6496/one-by-one/blob/main/%EC%8B%9C%EA%B3%84%EC%97%B4%EB%B6%84%EC%84%9D/images/PredictionCone.PNG?raw=true)
## 3.5 Roll-Forward Validation
- 시계열 자료는 횡단면 자료와 달리 training-validation partition을 random하게 뽑지 못하고 fixed할수 밖에 없음.
- 좀더 많은 validation 수행 위해 roll-forward 방법 사용.
- validation period의 시작점을 한 기간씩 앞으로 땡기면서 여러개의 partition 생성.
![rollfoward.PNG](https://github.com/jjlee6496/one-by-one/blob/main/%EC%8B%9C%EA%B3%84%EC%97%B4%EB%B6%84%EC%84%9D/images/rollfoward.PNG?raw=true)
- 일반적으로 roll-foward 방식이 fixed partioning 비해 성과가 우수하다.
