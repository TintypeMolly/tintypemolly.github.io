---
title: "Linear Regression"
date: 2020-03-26 12:13:00 +0900
category: dev
layout: post
tags:
  - dev
  - ml
---

## Tensorflow 2 Migration

제가 주 참고자료로 사용하려는 자료는 홍콩과기대 김성훈님이 만드신 [모두를 위한 딥러닝 강좌 시즌 1](https://www.youtube.com/watch?v=BS6O0zOGX4E&list=PLlMkM4tgfjnLSOjrEJN31gZATbcj_MpUm&index=1)이라는 강의입니다.
강의에 쓰이는 슬라이드 자료 등은 그분의 [블로그](http://hunkim.github.io/ml/)에서 찾을 수 있었습니다.

처음으로 걸림돌에 부딪힌 것은 아주 단순한 선형회귀의 Lab 강의였습니다.

<iframe
  width="560" height="315"
  src="https://www.youtube.com/embed/mQGwjrStQgg"
  frameborder="0"
  allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
></iframe>

해당 강의영상은 2017년 3월에 업로드 된 것이었고, 그 당시에는 Tensorflow의 버전이 1.x대였습니다.
현재 최신 안정버전은 2.1버전으로, 당시의 API와는 상당한 차이가 있었습니다.

1. `tf.Session` 사라짐
2. `tf.function` 사용 필요
3. 일부 요소의 패키지 위치 이동 (예: `tf.random_normal` -> `tf.random.normal`)

기존의 예제 코드를 참조하여 Tensorflow 공식 문서 중 [TF1에서 TF2로 이전](https://www.tensorflow.org/guide/migrate)을 참조하여 코드를 변경해보았습니다.

```python
import numpy as np
import tensorflow as tf

x = [1, 2, 3]
y = [3, 5, 7]

W = tf.Variable(tf.random.normal([1]), name='weight')
b = tf.Variable(tf.random.normal([1]), name='bias')


@tf.function
def hypothesis():
    return x * W + b


def cost():
    return tf.reduce_mean(tf.square(hypothesis() - y))


optimizer = tf.keras.optimizers.SGD(learning_rate=0.1)
for _ in range(2000):
    optimizer.minimize(cost, var_list=[W, b])


print(W, b)
```

## Keras

Tensorflow 2 버전에서는 Tensorflow의 저수준 API를 직접 사용하기 보다는 Wrapper인 Keras의 사용을 권장하고 있었습니다.
아직 Layer의 종류에 대해서는 모르는 것이 많지만,
레이어가 많아지면 좀 더 이해하기 쉬운 표현을 쉽게 쓸 수 있을 것 같습니다.

특히 머신 러닝 입문용으로 보았던 아래의 3Blue1Brown의 영상에서 설명했던 개념들이 그대로 나오고 있어서 어떤 인자를 어디서 초기화시키는지 잘 보이는 점이 좋은 것 같습니다.

<iframe
  width="560"
  height="315"
  src="https://www.youtube.com/embed/aircAruvnKk"
  frameborder="0"
  allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
></iframe>

Keras는 Tensorboard 등의 기타 도구들과도 쉽게 연동되고 있기 때문에
사용법을 좀 더 열심히 공부해서 익숙해지려고 합니다.

Keras를 사용해서도 비슷하게 Linear Regression 하는 코드를 작성해보았습니다.

```python
import numpy as np
import tensorflow as tf

x = [1, 2, 3]
y = [3, 5, 7]

model = tf.keras.Sequential(
    layers=[
        tf.keras.layers.Dense(1),
    ],
    name="Hypothesis"
)

model.compile(
    optimizer="sgd",
    loss="mean_squared_error",
)

model.fit(
    x, y, epochs=2000, verbose=0
)

model.evaluate([4, 5, 6], [9, 11, 13])

print(model.predict([7, 8, 9]))
```

한 가지 아쉬웠던 것은 weight와 bias 값을 직접 보는 방법을 찾지 못했던 것입니다.
볼 방법을 앞으로 공부하면서 찾아볼 생각입니다.
