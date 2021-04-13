---
layout: post
title:  "Introduce Basic Loss Function on Machine Learning"
date:   2021-03-31 15:28:11 +0530
categories: MSE MAE RMSE Cross-Entropy
use_math: true
---
🛠 기본적인 머신러닝 손실함수에 대하여

_____________________________________



# Loss Function

**Deep Learning**은 기본적으로 node 값을 가중치와 연산해서 다음 node로 보내는 연산이다. 

👉 수학적으로는 합성함수의 계산이다. ex) $f(g(h(x)))$

👉 가중치와 node는 선형결합의 형태이다. 따라서 node 연산만을 거치면 Linear하게 데이터를 표현하게 된다. 

👉 하지만 해결해야하는 대부분의 문제는 선형관계가 아니므로(더 복잡한 Non-Linear한 형태이므로) Activation function을 이용해 데이터를 nonlinear하게 표현하는 것이 필요하다.

## Loss Function이란?

Loss function은 위와 같은 연산을 통해 도출된 예측값과 정답을 비교하여 오차를 계산한다. 그리고 그 시점의 기울기를 구하여 업데이트한다.  이러한 과정에서 Global minimum 값에 도달하기에 적합 형태는 Convex function이다. 

convex function
$f(x)$ 내의 두 점 $x1, x2$를 직선($a$)으로 이었을 때, 두 점 사이의 모든 $f(x)$는 직선 $a$보다 작은 값을 갖게 된다.

지도학습에서 손실함수는 모델의 평가지표를 정의한다. 손실함수 없이는 모델의 파라미터를 구할 수 없다.  손실함수에 따라 최적화의 난이도가 다르고 최종적으로 얻는 모델 파라미터도 달라진다. 
따라서 구체적인 문제에 따라 적절한 손실함수를 선택해야한다.

### 👉 Loss Function을 다음과 같이 나타낼 수 있다.

- 훈련 샘플의 형식 : $(x_i, y_i)$
    - $x_i \in X$ 는 $i$번째 샘플의 feature를 나타낸다.
- 파라미터가 $\theta$인 모델을 다음과 같이 정의하자 : $f(\cdot,\theta):X \rightarrow Y$
    - 이때 $i$번째 샘플에 대한 모델 출력은 다음과 같다 : $f(x_i, \theta)$
- 위 모델과 샘플 레이블의 적합 정도를 나타내기 위한 손실함수는 다음과 같이 정의한다.
    - $L(f(x_i, \theta), y_i)$

  
### 👉 이진분류 문제 $Y = \{1, -1\}$에 대해 $L(f(x_i, \theta), y_i)$를 아래와 같이 만들 수 있다.

##### 1. 0-1

<center>$L_{0-1}(f, y) = 1_{fy \le 0}$</center>

  * $1_p$에서 $p$가 $True$이면 ($\le 0$ 이면) 1이고 아니면 0이 된다.
  * non-convex, non-smooth한 특성 때문에 Gradient Descent는 적용 못함.

#### 2. Hinge
<center>$L_{hinge}(f, y) = max\{0, 1-fy\}$</center>

  * SVM의 loss function으로 사용된다. $fy$가 1 이상이면 오차를 무시하고, 1 미만이면 오차가 크도록 유도한다.

#### 3. Logistic
<center>$L_{logistic}(f, y) = \log_{2}(1 + exp(-fy))$</center>
 
#### 4. Cross Entropy
<center>$L_{cross\ entropy}(f, y) = -\log_{2}({1+fy \over 2})$</center>

  * 분류문제에서 좋은 성능을 낸다. 자세한 내용은 아래에!

#### 5. Square
<center>$L_{square}(f, y) = (f-y)^2$</center>

  * convexity의 성질을 갖기 때문에 경사하강법을 통해 최적화가 가능하다.

#### 6. Absolute
<center>$L_{absolute}(f, y) = \left\vert{f-y}\right\vert$</center>

  * 절대값 손실함수는 학습 데이터의 이상치, 특이점에 대해 제곱 손실함수보다 robust하다는 특징을 갖는다. 하지만 y=x에서 미분 불가능하다.
 
#### 7. Huber
<center>$ L_{Huber}(f, y)=\begin{cases}(f-y)^2, &\left\vert{f-y}\right\vert \le \sigma \\ 2\sigma\left\vert{f-y}\right\vert - \sigma^2, & \left\vert{f-y}\right\vert > \sigma \end{cases}$</center>

  * Huber 손실함수는 오차가 임곗값($\sigma$, 전형적으로 1)보다 작을 때는 2차함수이고, 오차가 $\sigma$보다 클 때는 선형함수이다. 선형 함수는 square    loss보다 이상치에 덜 민감하고, 이차 함수는 absolute loss보다 수렴 속도가 빠르고 정확도가 높다.


Loss function은 기본적으로 convexity의 특성을 갖지만, feature들의 스케일이 매우 다르면 Global minimum을 찾는데 더 많은 비용이 소요된다. 때문에 학습 전 데이터의 Scale을 맞추는 것이 매우 중요하다.


## 기본적인 Loss Function의 종류

## Regression

### MSE (Mean Squared Error)

<center>$MSE={1\over{n}}\sum_{i=1}^{n}(\hat{y_i}-y_i)^2$</center>
  

MSE는 곡선에서 어떤 두 점을 선택해 선을 그어도 곡선을 가로지르지 않는 Convex Function이다. 따라서 Local minimum이 없고, 하나의 Global minimum만 존재한다.
또한 연속 함수이고 기울기가 급격하게 변하지 않기 때문에 경사하강법을 통해 Global minimum에 가깝게 접근할 수 있다는 것을 보장한다.

### MAE (Mean Absolute Error)

<center>$MAE={1\over{n}}\sum_{i=1}^n {|\hat{y_i}-y_i|}$</center>  

회귀 학습에 사용하는 손실함수에 일반적으로 MSE를 사용하지만 학습 데이터에 이상치가 많다면 MAE를 사용할 수 있다.
또는 MSE와 MAE를 조합한 Huber 손실을 사용할 수 있다. Huber 손실은 위에서도 언급했지만 특이점에 대해 robust한 특징을 가진다.

### RMSE

<center>$RMSE=\sqrt{\frac{1}{n}\sum_{i=1}^{n}(\hat{y_i}-y_i)^2}$</center>

RMSE와 MAE 모두 예측값의 벡터와 타겟값의 벡터 사이의 거리를 구하는 방법이다. RMSE의 경우 n개의 제곱근의 합으로 나타냄으로 유클리디안 거리에 해당한다. $l_2 norm$ 이라고도 하며 $\lVert \cdot \rVert_2$로 표시한다.

MAE는 절댓값의 합으로 나타냄으로 $l_1 norm$에 해당하며 $\lVert \cdot \rVert_1$로 표기한다. 맨해튼 거리에 해당한다.
$norm$의 지수가 클 수록 큰 값의 원소에 치우치며 작은 값은 무시되는 특성이 있다. 따라서 RMSE가 MAE보다 조금 더 이상치에 민감하다. 이상치가 없는 데이터에서는 RMSE가 잘 맞아 일반적으로 널리 사용된다.

## Classification

### Entropy

- Entropy는 정보이론에서 등장한 개념이다. 여기서 $y_i$는 정보가 등장하는 확률을, $\log{1 \over y_i}$는 information gain을 의미한다.
- 정보이론의 핵심 아이디어는 잘 일어나지 않는 사건, 희귀한 사건(unlikely event)은 자주 발생하는 사건보다 정보량이 많다(informative)는 것이다. 머신러닝에서는 데이터의 확률 분포의 특성을 알아내거나 확률 분포 간 유사성을 정량화하는 데 사용된다.
- 위 아이디어를 3가지로 표현하면 다음과 같다.
    - 자주 발생하는 사건은 낮은 정보량을 가진다. 발생이 보장된 사건은 그 내용에 상관없이 전혀 정보가 없다는 걸 뜻한다.
    - 덜 자주 발생하는 사건은 더 높은 정보량을 가진다.
    - 독립사건(independent event)은 추가적인 정보량(additive information)을 가진다. 예컨대 동전을 던져 앞면이 두번 나오는 사건에 대한 정보량은 동전을 던져 앞면이 한번 나오는 정보량의 두 배이다.

*Reference :* [정보이론 기초](https://ratsgo.github.io/statistics/2017/09/22/information/)

- 예시 : 용의자를 추정하는데 '용의자의 성이 '김'씨이다'는 정보와 '용의자의 성이 '곽'씨이다'는 정보 중 어느 쪽에 정보가 더 많은가? 

    👉 '곽'씨를 얻었을 때이다. 
즉, 내가 얻는 정보의 양이라는 것은 그 정보의 확률과 반비례한다. 따라서 $y_i$라는 정보 확률에 대해 information gain은 ${1 \over y_i}$인 것이다.

<center>$$H(y) = \sum_iy_i\log{1 \over y_i}$$</center>

### Cross Entropy

- Entropy가 정보에 대해 최적으로 인코딩, 압축하는 것이라면 Cross Entropy는 $Q$라는 잘못된 확률정보를 통해 얻은 값이다.
- 즉, 크로스 엔트로피는 정답 데이터의 분포 $P(x)$와 모델이 추정한 데이터 분포 $Q(x)$간의 차이를 최소화한다.

<center>$$H(p, q) = -\sum_xP(x)\log{Q(x)}$$</center>
  

### Binary Cross Entropy

<center>$$L = \sum_{i=1}^n(-y_i\log(\hat{y_i}) - (1-y_i)\log(1-\hat{y_i}))$$</center>
  

### Categorical Cross Entropy

<center>$$ L = -\frac{1}{N}\sum_{j=1}^{N}\sum_{i=1}^{C}t_{ij}log(y_{ij})$$</center>
  

그렇다면 왜 분류모델에서 Square Loss와 같은 Loss Function을 쓰지 않고 Cross Entropy를 쓰는 것일까? Binary Cross Entropy와 MSE를 아래와 같이 비교해보자.

ground truth를 $y = \{0, 1\}$, prediction을 $\hat{y}$인 분류문제를 적용해보면 아래와 같다.

<center>$$Mean \ Squared \ Error \ Loss \\
L = {1\over m}\sum(y_i-\hat y_i)^2 \\ L = (y - \hat y)^2$$</center>
  

<center>$$Binary \ Cross-entropy \ Loss \\
L = -{1 \over m}\sum[y_iln(\hat y_i) + (1-y_i)ln(1-\hat y_i)] \\
L = yln(\hat y) + (1-y)ln(1- \hat y)$$</center>
  

위 식을 각각 $y = 0$일 때와 $y = 1$일 때의 그래프를 그려보면 다음과 같다.

<center><img src="https://user-images.githubusercontent.com/59910975/114350767-9dd73280-9ba4-11eb-93bb-202593e8fc9e.png"></center>


가운데 $x$축을 기준으로 
위 두개 그래프는 $mse$, 
아래 두 개 그래프는 $cross \ entropy$이다.
$y = 0$일 때 $\hat y = 0.9$의 값이 나왔다고 가정해보자. 

**초록색**의 $cross \ entropy$ 그래프의 $0.9$ 지점 미분값(기울기)는 **파란색** $mse$그래프의 $\hat y = 0.9$ 지점 기울기보다 가파른 값을 가진다.

<center>$$\hat y = 0.9 \\
L_{SE} = 0.81 ,  \ L_{CE} = 2.3 \\ {} \\
{\partial{L_{SE}} \over \partial{\hat y}} = 1.8, \\
{\partial{L_{CE}} \over \partial{\hat y}} = 10.0
$$</center>
  

정답 라벨($y$)이 $0$일 때 예측값($\hat y$)이  $0.9$가 나왔다는 것은 모델이 정답과 전혀 다른 예측을 하고 있다는 것인데,
이런 경우 $mse$보다 $cross \ entropy$에서 $\hat y$ 업데이트를 큰 폭으로 해주므로 학습율이 더 좋아진다.

<center>$${\partial{L_{SE}} \over \partial{w}} = {\partial{L_{SE}} \over \partial{\hat y}}*{\partial{\hat y} \over \partial{\hat w}} , \\ = 1.8*{\partial{\hat y} \over \partial{\hat w}}\\ {} \\ 
{\partial{L_{CE}} \over \partial{w}} = {\partial{L_{CE}} \over \partial{\hat y}}*{\partial{\hat y} \over \partial{\hat w}} \\ 
= 10.0*{\partial{\hat y} \over \partial{\hat w}}$$</center>
  

즉, Loss Function에 대해 정리하면 다음과 같다.

- 파라미터 최적화를 위한 목적함수가 된다.
- Task 마다 적절한 형태로 사용되어야(만들어져야) 한다.
- Convexity는 Loss function이 하나의 global minimum을 갖게 해준다.
- 회귀에는 주로 MSE, MAE, RMSE를, 분류에는 Cross Entropy Loss Function을 사용한다.

*Reference:*

- [Why do we need Cross Entropy Loss?](https://www.youtube.com/watch?v=gIx974WtVb4&t=373s)
- Hands-On Machine Learning
- 데이터 과학자와 데이터 엔지니어를 위한 인터뷰 문답집