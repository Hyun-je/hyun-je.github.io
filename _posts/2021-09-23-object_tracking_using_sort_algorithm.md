---
layout: post
toc: true
title: "SORT 알고리즘을 이용한 오브젝트 추적"
categories: vision
tags: [vision, c++, kalman filter, sort]
---

![original](https://user-images.githubusercontent.com/7419790/161966056-04d2caad-f5a0-46e7-82e7-cb478ccf78c0.jpg)

객체 추적(Object Tracking)이란 영상의 각 프레임에서 인식된 객체들 중 동일한 객체가 서로 연관되어 있음을 판단하여 각 객체의 궤적(Trajectory)를 얻어내는 방법이다. 객체 인식(Object Detection)을 통해서는 하나의 프레임에 한정하여 객체의 공간상의 정보만을 얻을 수 있지만 객체 추적을 통해서는 시간의 흐름에 따라 변하는 정보까지도 얻을 수 있으므로 훨씬 더 다양한 응용을 가능하게 해준다.

객체 추적을 위한 알고리즘 중 SORT(Simple Online and Realtime Tracking)는 간단한 구조에 낮은 연산 성능을 요구한다. 따라서 실시간 처리에서 사용 가능하며 2D 이미지 뿐만 아니라 3D 포인트 클라우드 상의 객체 추적으로도 확장 가능한 범용성도 가지고 있다.

SORT는 영상 시퀀스의 각 프레임에서 객체 인식을 통해 얻은 객체의 위치 및 크기 정보를 입력으로 동작한다. 최근에는 CNN 계열의 객체 인식기의 성능이 어느 정도 보장되어 있으므로 이와 조합하여 높은 추적 성능을 얻을 수 있다.

## 알고리즘 구성
SORT 알고리즘은 현재 프레임에서 인식(Detection)한 오브젝트의 정보와 이전 프레임을 통해 예측(Prediction)한 오브젝트의 정보를 가지고 서로 연관성 높은 매칭하는 방식으로 이루어진다. 안정적인 성능을 얻기 위해서는 각 단계를 잘 이해하고 적절한 알고리즘을 선정하여야 한다.

### 1. Detection
인식 알고리즘은 적용하고자 하는 어플리케이션에 따라 간단한 Blob Detection 에서 부터 YOLO 까지 이미지 상에서 다중 객체에 대한 Bounding Box를 얻을 수 있는 것이라면 어떤 것이든 사용 가능하다. 안정적인 인식 성능이 바탕이 되어야 이후에 수행할 모든 하므로 세심하게 선정되어야 한다. 

### 2. Predection
예측은 이전 프레임으로부터 수집한 객체의 정보들을 이용하여 현재 프레임에서의 객체의 정보를 추정하는 과정이다.



어떤 Feature를 선정할 것인지 어떤 운동 모델을 사용할 것인지 세심하게 고려할 필요가 있다.


즉, 객체의 운동 모델을 선정하고 파라미터를 
기본적으로는 물체가 등속 운동을 하고 있는 것으로 가정하여 선형 칼만 필터(Kalman Filter)를 이용하는데 2D 이미지 상의 추적을 위해서는  객체 Bounding Box의 X, Y, Width, Height와 그 미분값을 상태값으로 놓을 수 있다. 최근에는 LSTM 기반으로 객체의 움직임을 추적하여 성능을 높인 사례도 있다.

경우에 따라서는 복잡한 추정 과정 대신 직전 프레임의 정보를 그대로 사용할 수도 있지만 그렇게 하는 경우 움직임이 빠른 물체는 매칭이 어렵게 되는데 이는 다음 단계인 Assignment 과정을 알고 나면 이해할 수 있다.

### 3. Assignment
할당은 Detection과 Predection을 통해 얻은 서로 연관된 객체를 매칭하는 것으로 SORT의 가장 핵심적인 단계이다. Detetction과 Predection으로 얻은 각 영역에서 겹치는 영역인 IOU(Intersection of Union)를 구하고 IOU의 크기에 따라 가장 적절한 쌍을 매칭 후 각 3가지의 상태를 분류한다.

## 결론

SORT 알고리즘의 구조는 비교적 단순하지만 칼만 필터 및 헝가리안 알고리즘에 대한 사전지식이 필요하다. 

 경우 직접 구현하기에는 다소 복잡하다. 따라서 환경에 따라 적절한 라이브러리를 활용하는 것이 좋다. 



[SORT-ros](https://github.com/Hyun-je/SORT-ros)


## 참고자료
- [Alex Bewley(2016) Simple Online and Realtime Tracking](https://arxiv.org/abs/1602.00763)
- [Wikipedia - Kalman filter](https://en.wikipedia.org/wiki/Kalman_filter)
- [Wikipedia - Hungarian algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm)