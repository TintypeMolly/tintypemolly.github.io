---
title: "Tensorflow 세팅"
date: 2020-03-21 15:01:00 +0900
category: dev
layout: post
tags:
  - dev
  - ml
---

## 요약

컴퓨터가 매우 좋은 것이 아니라면 Colab을 쓰는 것이 좋습니다.
Linux에서의 설정이 Windows보다 쉽습니다.

## ML Framework

Tensorflow는 Google 검색이나 Github stars 수를 봤을 때 가장 유명한 ML 프레임워크로 보입니다.
경쟁자로 pytorch 같은 것들이 있기는 하지만, 주변에 Tensorflow를 하는 사람이 많기도 해서 큰 고민 없이 Tensorflow를 골랐습니다.

## 버전 선정

Tensorflow를 NVIDIA GPU를 이용해서 실행하기 위해서는 CUDA와 cuDNN을 설치해야 합니다.
그런데 무턱대고 최신버전을 설치하면 안 된다는 것을 최신버전을 설치하고 나서야 알게 되었습니다.
ML을 공부하고 있는 후배님의 감사한 조언에 따르면 Tensorflow 버전을 먼저 고르고 거기에 맞는 CUDA와 cuDNN 버전을 설치하는 것이 가장 간편한 방법이라고 합니다.
Tensorflow의 버전에 따른 CUDA와 cuDNN 버전 테이블은 [Linux](https://www.tensorflow.org/install/source#linux), [Windows](https://www.tensorflow.org/install/source_windows#gpu)에 나와 있습니다.

제가 공부를 시작한 시점에서 사용한 버전은 Tensorflow 2.1, CUDA 10.1, cuDNN이 7.6 버전이었습니다.

## 설치

### Ubuntu

Ubuntu 등의 Linux의 경우 Docker를 활용하면 매우 편리한 것 같습니다.
CUDA를 직접 설치해본 적은 없지만 설치 과정이 다소 복잡한 것을 소문으로 전해들었습니다.
불필요한 패키지를 추가하기 싫었지만 어쩔 수 없이 CUDA 드라이버를 다운 받았습니다.
하지만 CUDA를 설치하기 직전에 동료의 도움으로 Docker를 사용할 수 있음을 발견했습니다.
[nvidia/cuda](https://hub.docker.com/r/nvidia/cuda) 같은 Docker 이미지를 활용하면 이미 CUDA가 설치된 환경을 얻을 수 있습니다.
Tensorflow까지 설치된 이미지가 필요하다면 [tensorflow/tensorflow](https://hub.docker.com/r/tensorflow/tensorflow):2.1.0-gpu-py3 등을 사용하면 되겠습니다.
다만

1. Ubuntu에 GPU 드라이버(예를 들어 우분투의 경우 `nvidia-driver-<버전>`)가 설치되어 있어야 하고
2. `docker run`을 할 때에 인자로 `--gpus <GPU 수>`를 줘야합니다.

준비가 되었다면 `docker run -it --rm --gpus 1 tensorflow/tensorflow:2.1.0-gpu-py3 /bin/bash` 같은 명령어로 해당 환경으로 들어갈 수 있습니다.

### Windows

Windows 머신에서는 docker를 사용해도 그 환경이 vm 속이기 때문에 CUDA를 사용할 수 있는 방법이 없었습니다.
그냥 홈페이지에서 CUDA와 cuDNN을 다운받아서 정직하게 설치했습니다.
CUDA는 CUDA만 설치하는 것이 아니라 절대로 쓸 일 없을 것 같은 잡다한 프로그램들을 설치하게 해서 고통스러웠습니다만 어쩔 수 없었습니다.
cuDNN의 경우는 조금 더 깔끔해서 압축을 풀고 적당한 곳에 놓아둔 뒤, dll을 가지고 있는 경로를 `PATH`에 추가해주면 됩니다.
저는 `%HOME%\cuda` 에 압축을 풀어두고 `%HOME%\cuda\bin`을 `PATH`에 추가했습니다.

## 설치 확인

MNIST를 돌려봤더니 잘 돌아갔습니다.

```python
import tensorflow as tf
from time import time


mnist = tf.keras.datasets.mnist
(x_train, y_train), (x_test, y_test) = mnist.load_data()
time_start = time()
x_train, x_test = x_train / 255.0, x_test / 255.0

model = tf.keras.models.Sequential([
    tf.keras.layers.Flatten(input_shape=(28, 28)),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5)

model.evaluate(x_test,  y_test, verbose=2)
time_finish = time()
print(time_finish - time_start)
```

## Colab

[Google Colab](https://colab.research.google.com/)은 Google이 무료로 제공하는 Jupyter 노트북 서비스고,
무료로 제공하는 클라우드 머신에서 GPU와 TPU도 사용할 수 있습니다.
위의 mnist 예제의 맨 윗 줄에 `%tensorflow_version 2.x`만 추가하고 돌려보니
제 GeForce GTX 1070을 장착한 윈도우 머신에서는 22초 걸린 작업을 28초만에 해냈습니다.
무료 제공이라 성능이 그다지 좋지 않을 거라고 생각했습니다만 틀렸던 것 같습니다.
물론 초기 로딩에 시간이 걸리고 IO 성능은 다소 낮은 것 같긴 합니다만,
세팅이 이미 다 되어 있는 완벽한 환경을 공짜로 클라우드로 제공한다는 점은 굉장히 매력적입니다.

## 결론

앞으로 간단한 예제는 Colab을 사용해서 돌리고, 오래 돌려야 하거나 무거운 작업은 로컬 머신에서 할 생각입니다.
