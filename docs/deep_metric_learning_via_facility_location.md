---
layout: page
group: "deep_metric_facility_location"
title: "Deep Metric Learning via Facility Location"
link_url: http://openaccess.thecvf.com/content_cvpr_2017/papers/Song_Deep_Metric_Learning_CVPR_2017_paper.pdf
---

## 개론

- 최근 연구에 따르면 CNN 을 활용하여 이미지의 유사도를 측정하고자 하는 시도가 많았음.
- 유사도 계산을 위한 방법을 모델 안에서 직접 학습하는 것을 Metric Learning 이라고 함.
- 지금까지 다양한 방법들이 제시되어 왔음.
- 이 논문에서는 앞서 사용되었던 방법들을 고찰하고 Facility Location 이라는 방법을 제안함.

![figure.1]({{ site.baseurl }}/images/{{ page.group }}/f01.png){:class="center-block" height="250px"}

## 관련 연구

- CNN을 활용한 연구 중 semantic embedding 을 활용하는 기법들을 순서대로 확인해보자.

### Contrastive Embedding

- 샴(Siamese) 네트워크를 활용한 방법. 한 쌍(pair)으로 이루어진 데이터 집합이 필요하다. \\( \\{({\bf x}\_{i},{\bf x}\_{j}, y\_{ij})\\} \\)
- Positive 샘플에 대해 거리를 0에 가깝게 하고, Negative 샘플에 대해서는 특정 threshold 이상의 거리를 유지하도록 학습
- Loss 함수는 다음과 같이 정의된다.

$$J = \frac{1}{m} \sum_{(i,j)}^{\frac{m}{2}} y_{i,j} D_{i,j}^{2} + (1-y_{i,j})\left[\alpha - D_{i,j}\right]_{+}^{2}\qquad{(1)}$$

- 여기에서 \\(f(\cdot)\\) 는 CNN 망으로부터 얻어진 *feature* 라고 정의한다.
- 그리고 두 *feature* 에 대한 거리는 \\(D\_{i,j} = \|\|\;f({\bf x}\_{i}) - f({\bf x}\_{j})\;\|\|\\) 로 정의한다.
- \\(y\_{i,j}\\) 는 *indicator* 로 \\(y\_{i,j} \in \\{ 0, 1\\}\\) 이며, 한 쌍의 데이터가 동일한 클래스이면 1, 아니면 0 의 값을 가지게 된다.
- \\([\cdot]\_{+}\\) 는 Hinge Loss 함수를 의미한다. 즉, \\(\max(0, \cdot)\\) 과 동일한 의미가 된다.
- Contrastive Embedding 방식의 장점과 단점
    - (장점) 비교적 쉽게 학습 집합을 구성해서 진행할 수 있다.
    - (단점) 학습 속도를 올리기 위해서는 학습 데이터 집합을 잘 선정해야 한다.
    - (단점) 모든 데이터에 대해 상수 margin \\(\alpha\\)를 선택해야 한다.
    - (단점) 클래스 단위이므로 시각적으로 다른 구조를 가지는 동일 클래스 데이터가 동일한 공간에 Embedding되게 된다.

### Triplet Embedding

- Triplet 방식은 Metric Learning 에서 가장 자주 쓰이는 기법이다. ([참고논문](https://arxiv.org/abs/1503.03832){:target="_blank"})
- 학습을 위해 3개의 쌍이 필요하다. \\( \\{({\bf x}\_{a}^{(i)},{\bf x}\_{p}^{(i)}, {\bf x}\_{n}^{(i)})\\} \\)
- 여기서 \\( ({\bf x}\_{a}^{(i)},{\bf x}\_{p}^{(i)}) \\) 은 같은 클래스에서 나온 쌍이고, \\( ({\bf x}\_{a}^{(i)}, {\bf x}\_{n}^{(i)}) \\) 는 다른 클래스에서 나온 쌍이다.
- Loss 함수는 다음과 같이 정의된다.

$$J = \frac{3}{2m}\sum_{i}^{\frac{m}{3}} \left[ D_{ia,ip}^{2} - D_{ia,in}^{2} + \alpha \right]_{+} \qquad{(2)} $$

- 여기서 \\(D\_{ia,jp} = \|\|\;f({\bf x}\_{i}^{a}) - f({\bf x}\_{i}^{p})\;\|\|\\) 이고 \\(D\_{ia,jn} = \|\|\;f({\bf x}\_{i}^{a}) - f({\bf x}\_{i}^{n})\;\|\|\\) 이다.
- \\([\cdot]\_{+}\\) 는 Hinge Loss 함수를 의미한다. 즉, \\(\max(0, \cdot)\\) 과 동일한 의미가 된다.

![figure.2]({{ site.baseurl }}/images/{{ page.group }}/f02.png){:class="center-block" height="150px"}

- Triplet Embedding의 장점 및 단점
    - (장점) Contastive Embedding 보다 더 좋은 성능을 낸다.
    - (장점) Loss 함수가 Convex 함수이다.
    - (장점) Embedding 공간이 임의로 왜곡되는 것을 허용함. (margin \\(\alpha\\) 는 상대적인 개념으로 적용됨)
    - (단점) 학습 시간이 오래 걸린다.
        - 학습이 진행되어 Converge 되면 Negative Margin 을 위반하는 학습 집합이 거의 등장하지 않게 되어 학습이 거의 진행되지 않는다.

- (참고) FaceNet 에서 사용하는  Semi-Hard Negative Sampling
    - Batch 단위 내에서 Anchor 샘플과 이에 대한 Positive, Negative 샘플을 추출함.
    - 이 때 학습이 잘 이루어질만한 Negative 샘플을 선정하여 사용.
    
$$n_{ap}^{*} = {\arg\min}_{n:D(a,n)>D(a,p)} D_{an} $$
    
### Lifted Structured Feature Embedding

- [관련논문 : Deep Metric Learning via Lifted Structured Feature Embedding](http://cvgl.stanford.edu/papers/song_cvpr16.pdf){:target="_blank"}
- 모든 positive pair가 모든 negative pair에 대해 거리를 비교하는 모델
- 이 논문은 정말 대충 보았으므로 틀리게 서술된 내용이 있을 듯함. (참고 바람)

![figure.3]({{ site.baseurl }}/images/{{ page.group }}/f03.png){:class="center-block" height="300px"}

- Loss 함수는 다음과 같이 정의된다.

$$J = \frac{1}{2|{\widehat P}|} \sum_{(i,j) \in {\widehat P}} \max{(0, J_{i,j})^{2}} \qquad{(3)}$$

$$J_{i,j} = \max{\left(\max_{(i,k) \in {\widehat N}}{(\alpha-D_{i,k})},\max_{(j,l) \in {\widehat N}}{(\alpha-D_{j,l})} \right)} + D_{i,j}$$

- 아이디어는 간단하다. 다만 몇 가지 문제가 존재한다.
- 문제
    - non-smooth.
    - All Pairs 에 대해 여러번 전체 연산을 수행해야 한다.
- 해결방안
    - non-smooth는 upper bound 식을 도입하여 해결한다. (관련 링크 : [LogSumExp](https://en.wikipedia.org/wiki/LogSumExp){:target="_blank"})
    - All Pairs 문제는 이전 연구에서 했던 것들과 유사하게 importance sampling 으로 푼다.
    
- Loss 함수를 다음과 같이 재정의한다.

$${\widehat J} = \frac{1}{2|P|} \sum_{(i,j) \in P}{ \max{\left( 0, {\widehat J}_{i,j}\right)^2}} \qquad{(4)}$$

$${\widehat J}_{i,j} = \log \left( \sum_{(i,k) \in N}{\exp{\left(\alpha-D_{i,k}\right)}} + \sum_{(j,l) \in N}{\exp{\left(\alpha-D_{j,l}\right)}} \right) + D_{i,j}$$

- 샘플링 방식은 간단하게 적자면,
    - 일단 positive pair 쌍들을 랜덤하게 선정 후,
    - 적당히 Batch 데이터 안에 포함하도록 구성한다.
    - 이후 Batch 내부에서 positive pair 에 포함된 각각의 샘플에 대해 hard negative 샘플을 구함.
    
![figure.4]({{ site.baseurl }}/images/{{ page.group }}/f04.png){:class="center-block" height="300px"}

- 최종적으로 다음과 같은 효과가 있다.

![figure.5]({{ site.baseurl }}/images/{{ page.group }}/f05.png){:class="center-block" height="300px"}
    
### N-Pair Loss

- [관련논문 : Improved Deep Metric Learning with Multi-class N-pair Loss Objective](http://www.nec-labs.com/uploads/images/Department-Images/MediaAnalytics/papers/nips16_npairmetriclearning.pdf){:target="_blank"}
- Triplet 구조를 일반화하여 (N+1)-Tuplet 네트워크를 제안함.
    - 1 Anchor, 1 Positive, (N-1) Negative Samples
    - N=2 인 경우 Triplet 과 동일한 모델이 된다.

![figure.6]({{ site.baseurl }}/images/{{ page.group }}/f06.png){:class="center-block" height="220px"}

- Loss 함수는 다음과 같다.

$$L(\{x,x^+,\{x_i\}_{i=1}^{N-1}  \};f) = \log\left( 1 + \sum_{i=1}^{N-1}{\exp(f^{T}f_i - f^Tf^+)}\right)$$

- 이 때 \\(f(\cdot)\\) 은 CNN 을 통과하여 얻은 feature 값이다. (논문에서는 Embedding Kernel 이라 표현)
- \\(\log(\cdot)\\) 항은 다음과 같이 풀어서 작성할 수 있다.

$$\log\left( 1 + \sum_{i=1}^{N-1}{\exp(f^{T}f_i - f^Tf^+)}\right) = -\log{\frac{\exp(f^Tf^+)}{\exp(f^Tf^+)+\sum_{i=1}^{L-1}{\exp(f^Tf_i)}}}$$

- 수식 구조를 보면 Multi-class Logistic Loss 즉, Softmax Loss와 그 모양이 유사하다.

#### 학습 집합의 구성

- 빠른 학습을 위해 효과적인 배치 구성이 필요하다.

![figure.7]({{ site.baseurl }}/images/{{ page.group }}/f07.png){:class="center-block" height="400px"}

(추가 필요)


## Facility Location

- 기존에 사용되던 Loss 들은 Mini-Batch 안에서의 Pairs/Triplets 로 정의되어 있음.
- 따라서 Global Structure 에 대한 정보는 고려되지 못함.
- 데이터 준비 과정도 매우 힘들다. (학습셋 구축이 가장 어려운 일이다.)
- 본 연구에서는 Clustering Quality Metric (NMI)로 바로 최적화 시키는 기법을 적용함.
