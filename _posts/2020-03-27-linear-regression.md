---
layout: post
title: "Linear Regression(선형 회귀) 경사하강법과 최소자승법으로 풀기"
tags: [MachineLearning]
author: jinhyo
feature-img: /assets/img/pexels/triangular.jpeg 
use_math: true
keywords: [LinearRegression, 선형회귀, 경사하강법, 최소자승법, OLS, GradientDescent]
---

## 개념 소개

선형 회귀(Linear Regression)은 하나의 종속변수 y와 하나 이상의 독립변수 x의 선형관계를 모델링하는 통계학 기법이다. x의 값이 변함에 따라서 y의 값이 종속적으로 변하는 관계가 있다고 가정한다. 통계학 용어를 기계학습 용어로 바꿔보자면 하나의 타겟과 하나 이상의 피처 사이의 선형관계를 분석하는 기법이라고 할 수 있다.  

<center>
<img src="https://user-images.githubusercontent.com/50395556/77673802-01a8b080-6fce-11ea-8935-b30daf86fb96.png">  
</center>  

그림에서 보이는 것처럼 피처에 따라서 타겟이 일정한 패턴으로 변화하는 양상을 보일 때 피처와 타겟 간에 선형관계가 있다고 한다. 이때 선형관계를 대표하는 선을 그을 수 있는데, 이 선을 선형 모델이라 한다.  

선형 모델은 100여 년 전에 개발됐지만, 현재도 널리 쓰이고 있는 기법이다. 차후에 학습할 딥러닝을 이해하기 위해서도 선형 회귀의 개념이 사용된다.

## 알고리즘 설명

앞에서 그림으로 보았던 선형회귀모델은 2차원 상의 일직선이므로 다음과 같은 다항함수로 표현할 수 있다.  

$$
\begin{align*} 
h(x) = wx + b 
\end{align*}
$$  

$h(x)$는 선형 회귀 모델이다. 통계학에서는 어떤 변수와 다른 변수 간의 관계에 대한 주장을 가설(hypothesis)이라고 한다. hypothesis에서 앞글자를 따서 h라고 주로 표기한다. $w$는 독립변수 $x$에 곱해지는 계수(coefficient)이다. 기계학습에서는 가중치(weight)라고 한다. $w$의 크기는 $x$가 결과에 미치는 영향력과 비례한다. 절편 $b$는 편향(bias)을 의미한다. 문헌에 따라 $w_0$라고 표기하기도 한다.  

$$
\begin{align*} 
\hat{y} = h(x) 
\end{align*}
$$  

모델에 특정한 값을 입력했을 때 얻어지는 결과를 라고 한다. $\hat{y}$는 모델이 만들어낸 예측값이다.

### ▷ Linear Regression 종류

- 단순 선형 회귀

$$
\begin{align*} 
h(x) = wx + b 
\end{align*}
$$  

단순 선형 회귀는 독립변수가 1개인 선형 회귀를 일컫는다. 위에서 봤던 그림은 피처가 1개인 단순 선형 회귀의 그래프였다. 이 포스팅에서는 알고리즘 이해를 돕기 위해 단순 선형 회귀를 기준으로 설명한다. 단순 선형회귀는 2차원으로 그릴 수 있기 때문에 직관적으로 이해하기가 쉬우며, 원리는 다중 선형 회귀와 같다.  

- 다중 선형 회귀

$$
\begin{align*}
h(x) = w_{1} x_{1} +w_{2} x_{2} +w_{3} x_{3} +...+ w_{n} x_{n} + b
\end{align*}
$$  

다중 선형 회귀는 피처가 2개 이상인 선형 회귀를 말한다. 선형 회귀는 피처가 하나일 때 2차원 상의 직선, 두 개일 때 3차원 상의 평면이 되며, 피처가 더 많아질 경우 다차원 공간에서의 초평면이 된다. 피처의 개수는 늘어나더라도 타겟은 여전히 1개임에 유의하자. 실제 세계에서 발생하는 문제 대부분은 여러 피처와 관계가 있다. 선형 회귀라고 하면 보통 다중 선형 회귀를 의미한다.  

### ▷ Cost function: MSE

- Cost Function

실제값 $y$와 예측값 $\hat{y}$의 차이를 오차(error)라고 한다. 실제값에서 예측값을 뺀 오차는 적을수록 예측이 정확하다는 뜻이다. 기계학습은 실제값과 예측값의 오차(error)를 최소화하는 $w$와 $b$를 찾아가는 과정이다. $w$, $b$와 오차의 관계를 함수로 나타낸 것을 비용함수(Cost function)라고 부른다. 동의어로 손실함수(Loss function), 목적함수(Objective function)라고 부르기도 한다.  

오차에 대해 더 자세히 이해하기 위해서 예를 들어 보자. 근무 시간을 기반으로 성과급을 예측하는 문제가 있다고 하자. 근무 시간과 성과급에 대한 데이터는 다음과 같다.  

<div align="center">
  <table>
    <thead>
      <tr>
        <th style="text-align: center">일한 시간</th>
        <th style="text-align: center">성과급</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td style="text-align: center">2</td>
        <td style="text-align: center">8</td>
      </tr>
      <tr>
        <td style="text-align: center">5</td>
        <td style="text-align: center">35</td>
      </tr>
      <tr>
        <td style="text-align: center">8</td>
        <td style="text-align: center">40</td>
      </tr>
      <tr>
        <td style="text-align: center">13</td>
        <td style="text-align: center">92</td>
      </tr>
      <tr>
        <td style="text-align: center">10</td>
        <td style="text-align: center">83</td>
      </tr>
    </tbody>
  </table>
</div>  

근무 시간은 피처이고 성과급은 타겟이다. $w$와 $b$를 임의의 숫자 5과 2라고 하면 $\hat{y}=5x+2$이고, 모델은 다음과 같이 나타낼 수 있다.  

<center>
<img src="https://user-images.githubusercontent.com/50395556/77673868-17b67100-6fce-11ea-8dfd-47e7c907ad29.png">
</center>  

주황색 선은 일한 시간이 주어졌을 때 모델의 예측값이다. 일한 시간과 성과급의 선형관계를 반영하고 있지만, 실제값과 예측값 사이에 오차가 있다. 실제값과 예측값 사이의 거리를 선으로 그어서 더 직관적으로 오차를 시각화해보았다.  

<center>
<img src="https://user-images.githubusercontent.com/50395556/77674019-4df3f080-6fce-11ea-91f9-6fae281219cf.png">
</center>  

오른쪽에 위치한 오차 2개가 매우 크다. 직선의 기울기($w$)를 더 가파르게 하거나 직선이 위쪽으로 평행이동하도록 절편($b$)을 조정하면 빨간색 직선을 줄일 수 있을 것 같다. $w$와 $b$를 조금씩 조정해보면서 빨간색 직선을 최대한 짧게 만드는 $w$와 $b$를 찾는 것이 기계학습의 목적이다.  

- MSE(Mean Squared Error)

오차를 전체적으로 줄여야 예측의 정확도가 높아진다. 전체적인 오차를 구하기에 앞서서 모든 오차를 직접 구해보면 다음과 같다.  

|피처(일한 시간, $x$)|실제값(성과급, $y$)|예측값(예측한 성과급 $\hat{h}$)|오차($y-\hat{y}$)|
|:---:|:---:|:---:|:---:|
|2|8|12|-4|
|3|35|27|8|
|8|40|42|-2|
|13|92|67|25|
|10|83|52|31|  

오차는 양수도 있고 음수도 있다. 전체적인 오차의 크기를 측정하기 위해서 오차를 모두 더했다가는 양수 오차와 음수 오차가 서로 상쇄되는 부작용이 발생한다. 전체적인 오차의 크기를 구하기 위해서 오차의 부호를 없애주어야 한다. 보편적으로 사용되는 방법은 모든 오차를 제곱한 후 평균을 내는 평균제곱오차(MSE, Mean Squared Errors)다. 식으로 나타내면 다음과 같다.  

$$
\begin{align*} 
MSE= \frac {1} {n} \sum _{i=1} ^{n} (y _{i} - {\hat{y _{i}}} ) ^{2} 
\end{align*}
$$  

오차를 제곱하거나 절대값을 씌우는 방법 등에 따라서 비용함수의 종류가 나뉜다. 절대값을 씌운 후 평균을 내는 MAE (Mean of Absolute Errors) 외에도 오차를 제곱 후 더해주는 SSE(Sum of Squared Errors), 오차에 절대값을 씌워 더하는 SAE(Sum of Absolute Errors) 등이 있다. 오차를 제곱하는 방법이 더 많이 사용된다. 제곱할 경우 양수든 음수든 오차가 클수록 제곱한 값은 더 커지기 때문에, 큰 오차가 발생했을 때 더 큰 페널티를 부여하는 효과가 있다. 성과급 예측모델의 평균제곱오차를 계산해보면 다음과 같다.  

$$
\begin{align*} 
\sum _{i=1} ^{n} (y _{i} - {\hat{y}} ) ^{2} =(-4) ^{2} +8 ^{2} +(-2) ^{2} +25 ^{2} +31 ^{2} =1670 
\end{align*}
$$  

$$
\begin{align*} 
\frac {1} {n} \sum _{i=1} ^{n} (y _{i} - {\hat{y}} ) ^{2} =1670/5=334 
\end{align*}
$$  

### ▷ Optimizer: Gradient Descent(경사하강법)

- Optimizer

비용함수를 최소화하는 $w$와 $b$를 구하는 최적화 알고리즘을 옵티마이저(Optimizer)라고 부른다.  

- Gradient Descent

경사하강법(Gradient Descent)은 가장 기본적인 옵티마이저이다. 비용함수의 기울기가 작아지는 방향으로 $w$와 $b$를 업데이트하는 방법이다. 먼저 비용함수를 그래프로 그리면 다음과 같다.  

<center>
<img src="https://user-images.githubusercontent.com/50395556/77674132-72e86380-6fce-11ea-895b-ba272d78bcab.png">
</center>  

비용(오차)은 실제값과 예측값의 차에 대한 제곱이었으므로 $w$를 x축으로 비용함수를 그려보면 아래로 볼록한 2차 함수의 모양이다. 모델의 오차가 오렌지색 점에 위치할 때, 오차가 가장 작으므로 최적화된 모델이 된다. 오렌지색 점에 해당하는 $w$값을 찾는 것이 기계학습의 목표이다. 반면 $w$가 최적화 값으로부터 멀어질수록 오차는 커진다. 만일 오차가 파란색 점에 해당하는 모델이 있다면, 이 모델은 최적화하기 위해서 $w$를 크게 하는 방향으로 업데이트해야 한다. 오차가 빨간색 점에 해당하는 모델이 있다면, 이 모델은 최적화하기 위해서 $w$를 작게 하는 방향으로 업데이트해야 한다. 각 점에 해당하는 모델을 직선으로 그어보면 다음과 같다.

<center>
<img src="https://user-images.githubusercontent.com/50395556/77674176-81367f80-6fce-11ea-8d2d-147c2b40a018.png">
</center>  

오렌지색 모델이 가장 선형관계를 잘 나타내고 있다. 오차도 확연히 줄어들어 보인다. 빨간색 모델은 오차가 더 커 보인다. 파란색 모델은 비용함수 그래프에서 최적화 지점에서 가장 멀었던 만큼, 오차도 가장 크다. $w$는 $x$의 계수이다. 즉, 1차 함수에서 기울기에 해당한다. 빨간색 모델은 최적화하기 위해서 기울기가 작아져야 하므로 $w$가 작아지게 업데이트해야 한다. 파란색 모델은 최적화하기 위해서 기울기가 커져야 하므로 $w$가 커지는 방향으로 업데이트해야 한다. 비용함수에서 $w$를 업데이트하는 방향에 대한 전략이 실제 모델의 최적화와 연결된다.  

<center>
<img src="https://user-images.githubusercontent.com/50395556/80851969-c69a3c80-8c5f-11ea-9dae-5926865d4182.png">
</center>  

다시 비용함수 그래프로 돌아와서 생각해보면, 모델의 오차가 그래프의 어느 지점에 있는지 일일이 시각화해서 업데이트하기는 쉽지 않다. 무엇보다 기계가 그래프를 보고 해석할 수는 없다. 경사하강법은 비용함수의 기울기를 이용해서 $w$를 업데이트할 방향을 찾아낸다. 미분을 이용해서 비용함수의 한 점에서의 접선의 기울기를 구할 수 있다. (모델 그래프에서의 기울기와 다름에 유의) 가장 오차가 적은 오렌지색 점에서의 특징은, 비용함수의 기울기가 0인 지점이라는 것이다. $w$가 최적화 값보다 작은 지점은 기울기가 음수이다. 파란색 점에서 미분했을 때 접선의 기울기가 음수이다. 기울기가 음수일 때는 가 커지는 방향으로 업데이트 한다. 반대로 $w$가 최적화 값보다 큰 지점에서는 접선의 기울기가 항상 양수이다. 빨간색 점과 같이 비용함수를 미분한 기울기가 양수인 모델은 $w$가 작아지는 방향으로 업데이트 한다. 비용함수를 미분한 접선의 기울기가 0이 될 때까지 반복한다. $w$를 업데이트하는 식은 다음과 같다.  


$$
\begin{align*} 
cost(w,b) &= \frac {1} {n} \sum_{i=1} ^{n} (y_{i} -h(x_{i} )) ^{2} \\ &= \frac {1} {n} \sum_{i=1} ^{n} (y_{i} -(x_{i} w + b)) ^{2} 
\end{align*}
$$  
  
$$
\begin{align*} 
w:=w- \alpha \frac {\partial}{\partial w} cost(w, b)
\end{align*}
$$  

$\frac {\partial} {\partial w} cost(w,b)$는 접선의 기울기이다. 기울기가 음수일 때 양의 방향으로 업데이트, 기울기가 양수일 때 음의 방향으로 업데이트하기 때문에 기존 $w$에서 빼준다. $\alpha$는 학습률(learning rate)이다. 지금까지는 업데이트의 방향에 대해서만 이야기했다. $\alpha$는 $w$를 얼마만큼 업데이트할지를 결정한다. $\alpha$가 크면 학습이 빨리 될 것 같지만, 무작정 크게 하면 $w$가 발산할 수 있다. 반면에 $\alpha$가 너무 작으면 학습 속도가 느려진다. 적정한 $\alpha$일 경우, 옵티마이제이션을 반복하다보면 접선의 기울기가 0인 지점에 수렴하게 된다. 접선의 기울기가 0에 가까워질수록 $\frac {\partial} {\partial w} cost(w,b)$가 작아지기 때문에 최적화에 근접할수록 $w$가 업데이트되는 크기가 작아지는 효과가 있기 때문이다. 적정한 $\alpha$를 찾는 것이 중요하며 learning rate($\alpha$)는 하이퍼 파라미터이다. 다음 그림은 $\alpha$가 너무 커서 $w$가 발산한 경우이다.  

<center>
<img src="https://user-images.githubusercontent.com/50395556/77674271-9f9c7b00-6fce-11ea-8a4d-b7c60ca9811b.png">
</center>  

지금까지는 $w$에 대해서만 설명을 했다. 하지만 실제로는 $b$에 대해서도 같은 옵티마이제이션 절차를 동시에 수행한다. 마찬가지로, 단순 회귀 모델이 아니라 다중 회귀 모델일 경우에도, 늘어나는 가중치($w_1, w_2, w_3, ... ,w_n$)에 대해서 옵티마이제이션을 동시에 수행한다.  

### ▷ Ordinary Least Square(최소자승법)

OLS(Ordinary Least Squares)는 RSS(Residual Sum of Squares)를 최소화하는 가중치 벡터를 행렬 미분으로 구하는 방법이다. 통계학에서는 실제값과 예측값의 오차를 residual이라고 부른다. 그러므로 RSS와 SSE(Sum of Squared Errors)는 같다.  


$$
\begin{align*}
\hat{y_{1}} =b+x_{1} w \\ \hat{y_{2}} =b+x_{2} w \\ \hat{y_{3}} =b+x_{3} w \\ \hat{y_{4}} =b+x_{4} w \\ \hat{y_{5}} =b+x_{5} w 
\end{align*}
$$  

성과급을 추정하는 문제는 연립 방정식으로 표현할 수 있고, 벡터와 행렬 연산으로 바꿀 수 있다. 그러므로 위의 단순 회귀 모델은 아래와 같이 표현할 수 있다.  

$$
\begin{align*}
{\hat{Y}} =X \theta
\space \space
(단,
\hat{Y} = \left( \begin{array}{c} {\hat{y_{1}}} \\ {\hat{y_{2}}} \\ {\hat{y_{3}}} \\ {\hat{y_{4}}} \\ {\hat{y _{5}}} \\ \end{array} \right), 
X= \left( \begin{array}{c} 1 x_{1} \\ 1 \ x_{2} \\ 1 \ x_{3} \\ 1 \ x_{4} \\ 1 \ x_{5} \\ \end{array} \right), 
\theta= \left( \begin{array}{c} b \\ w \\ \end{array} \right) 
)
\end{align*}
$$   

이때, 실제값 벡터와 예측값 벡터의 오차 벡터 e는 다음과 같다.  

$$
\begin{align*} 
e=Y- {\hat{Y}} =Y-X \theta 
\end{align*}
$$  

비용함수 SSE를 벡터로 표현하면 다음과 같다.  

$$
\begin{align*}
SSE & = e^{T} e \\
& = (Y-X \theta)^{T} (Y-X \theta) \\
& = Y^{T}Y -\theta^{T} X^{T} Y  - Y^{T} X \theta + \theta^{T} X ^{T} X \theta \\
& = Y^{T}Y - 2Y^{T} X \theta^{T}+ \theta^{T} X ^{T} X \theta
\end{align*}
$$  

$\theta^{T} X^{T} Y$와 $Y^{T} X \theta$는 벡터 내적 결과 같은 scalar 값이 되므로 하나로 정리할 수 있다. 비용함수를 최소로 하는 가중치 벡터 $\theta$를 구하기 위해서 비용함수를 $\theta$로 미분한 기울기 벡터는 다음과 같다. 기울기 벡터가 0일 때 비용함수가 최솟값을 가지므로 다음 식이 성립한다.  

$$
\begin{align*}
\nabla \theta(Y^{T} Y - 2Y^{T} X \theta + \theta ^{T} X^{T} X \theta) &= 0 \\
&= 2X^{T} X \theta - 2X^{T} Y
\end{align*}
$$  

가중치 벡터 $\theta$는 다음과 같이 구할 수 있다. $X^{T} X$의 역행렬이 존재해야 한다. $X^{T} X$의 역행렬이 존재하지 않을 경우에는 의사역행렬(pseudo inverse)을 이용해서 근사할 수 있다.  

$$
\begin{align*} 
\theta = (X^{T} X)^{-1} X^{T} Y 
\end{align*}
$$  

앞서 예를 들었던 성과급 데이터를 벡터로 표현하면 다음과 같이 나타낼 수 있다.  

$Y = \left( \begin{array}{c} 8 \\ 35 \\ 40 \\ 92 \\ 83 \\ \end{array} \right),
X = \left( \begin{array}{c} 1 \ 2 \\ 1 \ 5 \\ 1 \ 8 \\ 1 \ 13 \\ 1 \ 10 \\ \end{array} \right),
\theta = \left( \begin{array}{c} b \\ w \end{array} \right)$  

성과급 문제를 OLS로 풀어보자. $X^T X$와 $X^T Y$를 먼저 계산해보자.  

$$
X^T X = \left( \begin{array}{c} 
1 \ 1 \ 1 \ 1 \ 1 \\  
2 \ 5 \ 8 \ 13 \ 10 \\
\end{array} \right) 
\left( \begin{array}{c} 
1 \ 2 \\
1 \ 5 \\
1 \ 8 \\
1 \ 13 \\
1 \ 10 \\
\end{array} \right)
= \left( \begin{array}{c} 
5 \ 38 \\ 
38 \ 362 \\ 
\end{array} \right)
$$  

$$
X^T Y = \left( \begin{array}{c} 
1 \ 1 \ 1 \ 1 \ 1 \\  
2 \ 5 \ 8 \ 13 \ 10 \\
\end{array} \right) 
\left( \begin{array}{c} 
8 \\
35 \\
40 \\
92 \\
83 \\
\end{array} \right)
= \left( \begin{array}{c} 
258 \ 2537 \\ 
\end{array} \right)
$$  

SSE를 최소화하는 가중치 벡터를 구하는 식에 대입해보면 다음과 같다.  

$$
\begin{align*}
\theta &= (X^T X)^(-1) X^T Y \\
&= {\left( \begin{array}{c} 
5 \ 38 \\
38 \ 362 \\ 
\end{array} \right)}^{-1} 
\left( \begin{array}{c} 
258 \ 2537 \\ 
\end{array} \right) \\
&= \left( \begin{array}{c} 
-8.22404372 \ 7.8715847 \\ 
\end{array} \right)
\end{align*}
$$  

OLS로 구한 최적화 $\theta$를 $w$와 $b$에 대입해서 모델을 그려보면 다음과 같다. 오차를 최소화한 선형 회귀 모델이다.  

<center>
<img src="https://user-images.githubusercontent.com/50395556/77674357-cf4b8300-6fce-11ea-91f5-898044355c7e.png">
</center>
