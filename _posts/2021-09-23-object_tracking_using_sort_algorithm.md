---
layout: post
toc: true
title: "SORT 알고리즘을 이용한 오브젝트 추적"
categories: vision
tags: [vision, c++, kalman filter, sort]
---

객체 추적(Object Tracking)이란 영상의 각 프레임에서 인식된 객체들 중 동일한 객체가 서로 연관되어 있음을 판단하여 각 객체의 궤적(Trajectory)를 얻어내는 방법이다. 객체 인식(Object Detection)을 통해서는 하나의 프레임에서의 공간상의 정보만을 얻을 수 있지만 객체 추적을 통해서는 연속된 여러 프레임에서 나타나는 시간 흐름에 따라 변하는 정보까지도 얻을 수 있으므로 훨씬 더 다양한 응용을 가능하게 해준다.

![Object Tracking](https://user-images.githubusercontent.com/7419790/162551308-1ddcf8fc-f023-43e2-8953-19eb8e41467e.jpg)

실제로 비전 기술을 이용하여 무언가 실용적인 솔루션을 구현하다보면 객체 인식만으로는 한계가 있는 경우가 많고 결국 객체 추적을 필요로 하게 된다. 또한 객체 인식을 보조하여 실시간 처리 성능을 높이고 물체가 겹치고 가려지는 등의 상황에서도 적절한 대응을 가능하게 할 수도 있다.

객체 추적을 위한 알고리즘 중 SORT(Simple Online and Realtime Tracking)는 간단한 구조에 낮은 연산 성능을 요구하여 실시간 처리에서 사용 가능하다. 또한 2D 이미지 뿐만 아니라 3D 포인트 클라우드 상의 객체 추적으로도 확장 가능한 범용성도 가지고 있어 활용가치가 높다.

## 알고리즘 구성
SORT 알고리즘은 현재 프레임에서 인식(Detection)한 오브젝트의 정보와 이전 프레임을 통해 예측(Prediction)한 오브젝트의 정보를 가지고 서로 연관성 높은 매칭하는 방식으로 이루어진다. 안정적인 성능을 얻기 위해서는 각 단계를 잘 이해하고 적절한 알고리즘을 선정하여야 한다.

![SORT Diagram](https://user-images.githubusercontent.com/7419790/162550200-4f3fc1c4-1f8c-4fed-ae0f-b1c7616b1024.png)

### 1. Detection
SORT는 영상 시퀀스의 각 프레임에서 객체 인식을 통해 얻은 객체의 위치 및 크기 정보를 입력으로 동작한다. 인식 알고리즘은 적용하고자 하는 어플리케이션에 따라 간단한 영상처리 기반 Blob Detection 또는 딥러닝 기반 R-CNN / YOLO 등, 이미지 상에서 다중 객체에 대한 Bounding Box를 얻을 수 있는 것이라면 어떤 것이든 사용 가능하다.

인식 성능은 종합적인 추적 성능에서 큰 비중을 차지하며 대개의 경우 연산량도 가장 많은 편이다. 최근에는 CNN 계열의 인식 모델의 성능이 어느 정도 보장되어 있으므로 적당한 것을 선택하기만 하면 된다.

### 2. Estimation Model (Prediction)
예측은 이전 프레임으로부터 수집한 객체의 정보들을 이용하여 현재 프레임에서의 객체의 정보를 추정하는 과정이다. 어플리케이션의 특성에 따라 적절한 추정 모델을 만드는 것은 SORT 알고리즘의 구성 요소 중 가장 세심하게 선정하고 조율되어야 하는 단계이다. 

![kalman_filter](https://user-images.githubusercontent.com/7419790/162554795-dc3d3714-0b55-4417-9eca-e7ce1dea62c7.jpg)

단순하게는 객체에 등속 운동 모델을 적용하여 선형 칼만 필터(Linear Kalman Filter)를 이용하는데 예를 들어 객체 Bounding Box의 X, Y, 면적, 종횡비와 그 미분값을 상태값으로 놓을 수 있다. 단, 물체의 운동이 복잡하거나 서로 겹치고 가려지는 등 인식 조건이 열악한 경우 원하는 추적 성능을 달성하기 위해 더 정교한 모델이 필요할 수도 있다. 최근에는 LSTM 기반으로 모델을 추정하는 형태로 발전하고 있다.

### 3. Data Association (Assignment)
할당은 Detection과 Predection을 통해 얻은 서로 연관된 객체를 매칭하는 단계이다. 먼저 Detetction과 Predection으로 얻은 각 영역에서 겹치는 영역인 IOU(Intersection of Union)를 구한다.

![IOU](https://user-images.githubusercontent.com/7419790/162555259-163ec5a6-c7cf-438a-9dd6-e66d626e9f36.png)

그리고 IOU의 크기에 따라 가장 적절한 쌍을 매칭하여 매칭된 쌍과 매칭되지 않은 나머지로 객체를 분류한다. 이 때 헝가리안 알고리즘(Hungarian Algorithm)이 사용되는데 이 알고리즘은 $$O(n^3)$$의 상당히 높은 시간 복잡도를 가지므로 입력되는 객체의 수는 적절하게 제한될 필요가 있다.

### 4. Tracking
|매칭 결과|분류 기준|Tracker 처리|
|---|---|---|
|Matched Tracks|Detections 및 Predicions 에 모두 포함|유지|
|Unmatched Tracks|Predections 에만 포함|제거|
|Unmached Detections|Detections 에만 포함|생성|

매칭의 결과로 각각의 모든 객체의 상태는 위와 같이 3가지로 분류된다. 이 때 Unmatched Tracks와 Detections를 처리하는 전략은 필요에 따라 조정할 수 있다.

예를 들어 Unmatched 상태의 객체를 곧바로 제거하거나 생성하지 않고 일정 시간 상태를 유지하는 경우에 한하여 제거하거나 생성하는 등의 방법을 사용할 수 있다. 그 결과 Occlusion 객체에 대한 추적을 가능하게 한다거나 False-Positive를 감소 시킬 수 있다.

## 결론
SORT 알고리즘은 몇 가지 이미지 인식 알고리즘을 다룰 줄 알고 칼만 필터 대한 사전지식만 있으면 비교적 쉽게 구현하고 적용할 수 있다. 또한 각자의 어플리케이션에 맞게 적절한 Detection / Prediction 알고리즘을 조합할 수 있으므로 확장성도 뛰어나다고 할 수 있다.

SORT를 기반으로 Data Association를 좀 더 고도화 시킨 Deep SORT와 같은 시도도 있고 최근에는 딥러닝 기반 모델을 적용하여 추적 성능을 높이는 형태로 발전하고 있다.

## 참고자료
- [Alex Bewley(2016) Simple Online and Realtime Tracking](https://arxiv.org/abs/1602.00763)
- [GitHub - abewley/sort](https://github.com/abewley/sort)
- [Wikipedia - Kalman filter](https://en.wikipedia.org/wiki/Kalman_filter)
- [Wikipedia - Hungarian algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm)