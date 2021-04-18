# Activation function

## Why need Activation function?

Loss Function에서 설명했듯이 기본적인 인공신경망은 선형 결합 형태로 되어있다. (ex. $f(g(h(x)))$). node 값들에 가중치를 곱하고 더하는 형태이기 때문에 이전 Layer와 다음 Layer의 관계 또한 선형 결합이다. Layer 사이에 비선형성(nonlinear)을 추가해주지 않으면 아무리 Layer를 깊게 쌓아도 하나의 Layer와 동일해진다. 
따라서 복잡한 문제를 풀기 위해 Activation function을 사용하여 비선형성을 추가한다.

이론적으로 비선형 활성화 함수가 있는 충분히 깊은 신경망은 어떤 연속 함수(continuous function)도 근사할 수 있다.

Activation Function은 종류가 많고, 해결하고자 하는 Task에 따라 튜닝을 해주어야 할 때도 있다.

- Sigmoid — $\sigma{x} = {1 \over 1+e^{-x}}$
- tanh — $tanh(x)$
- ReLU — $max(0, x)$
- Leaky ReLU — $max(ax, x)$
- Maxout — $max(w_1^T + b_1, w_2^Tx + b_2)$
- ELU — $\begin{cases} x & x \ge 0 \\ \alpha(e^x-1) & x < 0 \end{cases}$

신경망을 훈련할 때, 한 Epoch이 끝나고 Loss Function을 기준으로 모든 파라미터가 업데이트된다. 그렇기 때문에 신경망 학습은 Loss 자체의 최적화를 푸는 문제로도 볼 수 있다. 
Activation Function을 사용해 NonLinear한 데이터 분포가 학습되면 Loss Function의 convex한 특성이 깨질 수 있다. 따라서 어떤 Activation function을 사용하느냐가 수렴성이나 학습 속도에 영향을 미치게 된다. 

### 1. Sigmoid

$$\sigma{x} = {1 \over 1+e^{-x}}$$

미분하면 다음과 같다.

$$f'(x) = f(x)(1-f(x))$$

함수값이 0과 1 사이에 있다는 것과 미분가능하다는 것이 장점이다.
output이 0과 1로 표현되기 때문에 주로 Classification 문제에서 마지막 레이어에 많이 사용된다.
또한 0과 1사이로 표현되었기 때문에 확률 개념으로 사용할 수도 있다.

![sigmoid](https://user-images.githubusercontent.com/59910975/115139255-c57a3f00-a06b-11eb-94cf-a5068086ff40.png)

Sigmoid 함수는 back propagation을 적용하면 취약할 수 있다. 미분함수를 보면 x가 매우 크거나 매우 작을 때 모두 0에 근사하게 되는데 이는 Gradient Vanishing 현상을 일으킨다.

이는 학습 연산 중에 0 값이 많이 곱해지므로 네트워크가 죽는 문제를 의미한다. 
다음은 Numpy로 구현한 Sigmoid 함수이다.

```python
import matplotlib.pyplot as plt
import numpy as np

# sigmoid function
t = np.linspace(-10, 10, 100)
sig = 1 / (1 + np.exp(-t))

# draw sigmoid graph
plt.figure(figsize=(8,6))
plt.axhline(y=0, color="black", linestyle="--")
plt.axhline(y=0.5, color="black", linestyle=":")
plt.axhline(y=1.0, color="black", linestyle="--")
plt.axvline(color="grey")
plt.plot(t, sig, linewidth=2, label=r"$\sigma(t) = \frac{1}{1 + e^{-t}}$", color="CornflowerBlue")
plt.xlim(-10, 10)
plt.xlabel("t")
plt.legend(fontsize=14)
plt.title("Sigmoid Activation function")
plt.show()
```

### 2. Hyperbolic tangent

$$tanh(x) = {sinh(x) \over cosh(x)} = {e^{2x}-1 \over e^{2x}+1}$$

미분하면 다음과 같다.

$$f'(x) = 1-(f(x))^2$$

Sigmoid 함수처럼 tanh 함수도 S자 모양이고 연속적이며 미분가능하다.
사실상 Sigmoid 함수를 평행 이동한 것과 같다.

![tangent](https://user-images.githubusercontent.com/59910975/115139256-c57a3f00-a06b-11eb-8634-d0d4a6cd1d71.png)

출력 범위는 -1과 1 사이이다. 따라서 훈련 초기에 각 층의 출력을 원점 근처로 모으는 경향이 있어 빠르게 수렴하도록 해주는 특징이 있다. 

RNN이나 LSTM 등 Recurrent Neural Net에서 잊혀져야 하는 값과 기억해야 하는 값을 결정하기 위한 가중치 의미로도 사용된다.

```python
#tangent function
t = np.linspace(-10, 10, 100)
tanh = (np.exp(2*t)-1) / (np.exp(2*t)+1)

# draw tangent graph
plt.figure(figsize=(8,6))
plt.axhline(y=-1.0, color="black", linestyle="--")
plt.axhline(y=0, color="grey")
plt.axhline(y=1.0, color="black", linestyle="--")
plt.axvline(color="grey")
plt.plot(t, tanh, linewidth=2, label=r"$\tanh(t) = \frac{e^{2t}-1}{e^{2t}+1}$", color="darkslateblue")
plt.xlim(-10, 10)
plt.xlabel("t")
plt.legend(fontsize=14)
plt.title("Hyperbolic Tangent Activation Function")
plt.show()
```

### 3. ReLU

$$ReLU(x) = \begin{cases}x & x>0 \\ 0 & x\le 0 \end{cases}$$

미분하면 다음과 같다.

$$f'(x) = \begin{cases}1 & x>0 \\ 0 & x \le 0 \end{cases}$$

ReLU함수는 0을 기준으로 양쪽이 선형함수이기 때문에 Sigmoid보다 수렴 속도가 빠르다는 장점이 있다. 양쪽 그래프를 미분하면 0 또는 1이기 때문에 0보다 큰 값을 다음 Layer로 그대로 내보내는 형태가 된다. 

![relu](https://user-images.githubusercontent.com/59910975/115139252-c4491200-a06b-11eb-902f-311a7d597cc5.png)

ReLU 함수는 이미지 연산을 할 때 유리한데, 이미지는 0~255 사이의 RGB 값으로 이루어진 데이터이기 때문에 음수가 나왔을 때 그 값에 0을 곱해 다음 Layer로 넘어가지 못하게 막는다. 

또한 ReLU는 Loss Function의 convexity를 어느 정도 보장한다. x=0을 제외하고는 Linear한 성질을 갖고 있기 때문이다. 

대표적인 단점으로 훈련하는 동안 일부 Layer가 0 이외의 값을 출력하지 않는 경우(이를 죽었다고 표현해서)인 Dying ReLU가 있다. 특히 큰 Learning rate을 사용했을 때 신경망의 Layer 절반이 죽어있기도 하다.

👉 신경망의 가중치가 음수로 바뀌어 Layer에서 내보내는 값이 음수 합이 되면 ReLU 함수의 그래디언트가 0이 되므로 경사하강법이 더이상 진행되지 않게 된다.

다른 단점으로는, 양수 가중치가 곱해진다고 해도 이전 Layer의 값에 ReLU 그래디언트 값 1이 곱해져 그대로 전달되기 때문에 큰 값이 연속해서 연산 될 수 있다는 위험이 있다.

```python
# relu function
t = np.linspace(-10, 10, 100)
relu = np.maximum(t, 0)

# draw relu graph
plt.figure(figsize=(8,6))
plt.axhline(y=0, color="grey")
plt.axvline(color="grey")
plt.plot(t, relu, linewidth=2, label=r"$ReLU(x)=max(0, t)$", color="lawngreen")
plt.xlim(-10, 10)
plt.xlabel("t")
plt.legend(fontsize=14)
plt.title("ReLU Activation Function")
plt.show()
```

### 4. Leaky ReLU

$$LeakyReLU(x) = \begin{cases}x & x\ge0 \\ scale*x & x< 0 \end{cases}$$

Leaky ReLU는 Dying ReLU를 해결하기 위한 방법으로 고안되었다.
$LeakyReLU_a(z) = max(scale*z, z)$로 정의되며 $scale$은 함수가 새는 정도(Leaky)를 결정한다. 일반적으로 0.1 혹은 0.01로 설정한다. 

이 기울기값은 LeakyReLU가 죽지 않게 (0 그래디언트가 곱해지지않게) 만들어준다. 

![leaky_relu](https://user-images.githubusercontent.com/59910975/115139251-c4491200-a06b-11eb-9b93-f2aa6076fee1.png)

[Bing Xu et al., 2015](https://arxiv.org/pdf/1505.00853.pdf) 에서 저자는 여러 ReLU 계열의 함수를 컨볼루션 연산에 대해 비교한 결과 LeakyReLU가 ReLU보다 항상 성능이 높다는 결론을 내렸다. $scale$ 값을 $0.2$로 두어 $x<0$ 일 때 더 큰 값을 통과하게 하는 것이 $scale=0.01$일 때보다 더 좋은 성능을 보였다.

위 논문에서는 주어진 범위에서 $scale(a)$값을 랜덤하게 선택하여 훈련 후, 테스트할 때 $a$값의 평균을 내어 사용하는 RReLU(randomized leaky ReLU)와 
$a$값을 하나의 파라미터로 두고 함께 학습하여 업데이트하는 PReLU(parametric leaky ReLU)도 함께 실험했다.

![relus](https://user-images.githubusercontent.com/59910975/115139253-c4e1a880-a06b-11eb-9a3b-f46a13df861f.png)
[https://arxiv.org/pdf/1505.00853.pdf](https://arxiv.org/pdf/1505.00853.pdf)

결과적으로 PReLU의 성능이 항상 좋았고, Leaky ReLU와 RReLU 역시 ReLU보다 좋은 결과를 냈다고 한다.

다만, 작은 데이터셋을 사용했을 때는 RReLU가 일종의 규제 역할을 해주는 것으로 보였으며, PReLU의 경우 작은 데이터셋에서 과적합되는  위험이 있었다.

```python
# leaky relu function
t = np.linspace(-10, 10, 100)
scale = 0.1
leaky_relu = np.maximum(t, t*scale)

# draw leaky relu graph
plt.figure(figsize=(8,6))
plt.axhline(y=0, color="grey")
plt.axvline(color="grey")
plt.plot(t, leaky_relu , linewidth=2, label=r"$LeakyReLU(x)=max(scale*t, t), scale=0.1$", color = "lightseagreen")
plt.xlim(-10, 10)
plt.xlabel("t")
plt.legend(fontsize=14)
plt.title("Leaky ReLU Activation Function")
plt.show()
```

### 5. ELU

$$ELU_a(x) = \begin{cases}x & x \ge 0 \\ \alpha(e^x-1) & x < 0 \end{cases}$$

[Djork-Ame Clevert et al, 2016](https://arxiv.org/pdf/1511.07289.pdf) 에 의해 제안된 ELU는 다른 모든 ReLU 계열 활성 함수보다 높은 성능을 기록했다. 훈련 시간이 줄었을 뿐 아니라 Test 데이터에서의 성능도 더 높았다. 

위 식에서 $a$는 $x$가 큰 음수값일 때 ELU 함수가 수렴하는 값을 정의한다. 

일반적으로 $a=1$로 설정하지만 하이퍼 파라미터처럼 변경하여 사용할 수 있다. $x < 0$일 때에도 ELU의 그래디언트가 0이 아니므로 Layer가 죽지 않게 한다. 

특히 $a = 1$일 때 ELU의 도함수는 $x=0$에서 연속적이 되어 ELU는 전체 구간에서 미분가능하게 된다. 

모든 구간에서 미분 가능하면 경사하강법의 속도가 높아진다.

![elu](https://user-images.githubusercontent.com/59910975/115139250-c3b07b80-a06b-11eb-8bcc-9bdd09e88fd6.png)

ELU 함수의 단점은 ReLU 계열 함수에 비해 연산 속도가 느리다는 것이다. 

$z < 0$인 구간에서 지수함수를 사용하기 때문인데, 훈련 시에는 수렴 속도가 빨라서 지수함수의 느린 계산이 상쇄되지만 Test 데이터에서는 ReLU 함수보다 연산속도가 더 느리다.

```python
# elu function
t = np.linspace(-10, 10, 100)
alpha = 1
elu = np.where(t > 0, t, alpha * (np.exp(t) - 1))

# draw elu graph
plt.figure(figsize=(8,6))
plt.axhline(y=0, color="grey")
plt.axvline(color="grey")
plt.plot(t, elu, linewidth=2, label=r"$ELU_a(x), \alpha=1$", color="mediumslateblue")
plt.xlim(-10, 10)
plt.xlabel("t")
plt.legend(fontsize=14)
plt.title("ELU Activation Function")
plt.show()
```

[Gunter Klambauer et al., 2017](https://arxiv.org/pdf/1706.02515.pdf) 에서는 이러한 ELU의 스케일을 조정하여 더 나은 성능을 내는 SELU(Scaled ELU) 활성 함수를 소개한다. 

$$SELU(x) = \lambda \begin{cases} x & x > 0 \\ \alpha({e^x} - 1) & x \le 0 \end{cases}$$

SELU는 Fully Connected Layer에 사용될때 네트워크가 Self-normalized 된다는 특징을 갖는다. 

Self-normalized 되면 학습하는 동안 각 Layer의 출력이 0의 평균과 표준편차 1을 유지하기 때문에 그래디언트 소실, 폭주의 위험을 방지한다.
그 결과 SELU는 깊은 신경망에서 다른 활성 함수보다 뛰어난 성능을 기록하게된다.

![selu](https://user-images.githubusercontent.com/59910975/115139254-c4e1a880-a06b-11eb-9fca-5f6e82094c67.png)

그러나 네트워크가 Self-normalize 되기 위한 조건이 있다.

- 입력 feature 역시 정규화되어야 한다. (평균 0, 표준편차 1)
- 모든 은닉층의 가중치는 르쿤 초기화(평균이 0이고 분산이 $\sigma^2 = {1 \over fan_{in}}$인 정규분포) 되어야 한다.
- 순환 신경망(Recurrent Neural Net)이나 Skip-Connection과 같이 순차적으로 연산하지 않는 네트워크 구조는 Self-normalize를 보장하지 않는다.

  
  
☝ 네트워크가 Self-normalize 되지 않으면 다른 활성함수보다 성능이 낮을 수 있다.

☝ 컨볼루션 신경망에서도 SELU가 효과 있다고 한다.

```python
# selu function
t = np.linspace(-10, 10, 100)
# alpha, scale from https://www.tensorflow.org/api_docs/python/tf/keras/activations/selu
alpha = 1.67326324
scale = 1.05070098
selu = scale*np.where(t > 0, t, alpha * (np.exp(t) - 1))

# draw selu graph
plt.figure(figsize=(8,6))
plt.axhline(y=0, color="grey")
plt.axvline(color="grey")
plt.plot(t, selu, linewidth=2, 
         label=r"$SELU(x), \alpha > 1, scale > 1$", color="mediumseagreen")
plt.xlim(-10, 10)
plt.xlabel("t")
plt.legend(fontsize=14)
plt.title("SELU Activation Function")
plt.show()
```

지금까지의 ReLU 계열 함수를 한번에 비교해보면 다음과 같다.

![all_relus](https://user-images.githubusercontent.com/59910975/115139249-c27f4e80-a06b-11eb-9862-5f8d1ed4ae28.png)

  
    
      

## 어떤 활성함수를 사용해야 할까?

일반적으로 좋은 성능을 기록하는 활성함수는
SELU > ELU > Leaky RELU 계열 함수들 > ReLU > tanh > sigmoid 순이다.

✅ 하지만 네트워크가 Self-normalize를 보장하지 않는 구조라면 ELU를 사용하는 것이 좋다.

✅ 추론 속도가 중요한 네트워크를 설계한다면 LeakyReLU를 고려한다.

✅ 데이터셋이 소규모이고, 신경망이 과적합 되었다면 RReLU로 학습을 규제할 수 있다.

✅ 데이터셋이 대규모라면 PReLU를 포함하는 것이 좋다.

✅ ReLU 활성 함수는 가장 널리 사용되므로 많은 라이브러리와 하드웨어 가속기는 ReLU에 최적화되어 있다. 학습 속도가 중요하다면 ReLU를 권장한다.

  
    
      
      
Reference:

- [MathWorks](https://www.mathworks.com/help/deeplearning)
- Hands-On Machine Learning
- 데이터 과학자와 데이터 엔지니어를 위한 인터뷰 문답집
- [Bing Xu et al., 2015](https://arxiv.org/pdf/1505.00853.pdf)
- [Gunter Klambauer et al., 2017](https://arxiv.org/pdf/1706.02515.pdf)
- [Djork-Ame Clevert et al, 2016](https://arxiv.org/pdf/1511.07289.pdf)
- [https://ml-cheatsheet.readthedocs.io/en/latest/activation_functions.html](https://ml-cheatsheet.readthedocs.io/en/latest/activation_functions.html)