---
layout: post
toc: true
title: "OpenCV 테니스 코트 인식 2 (카메라 자세 추정)"
categories: vision
tags: [opencv, python, vision]
---

카메라 모델에 대한 이해와 선형 대수에 대한 사전지식이 필요하므로 

카메라 자세 추정 기술은 2D의 이미지와 3D의 실제 공간을 연결해주는 수학적 도구로 최근 관심이 높아진 AR이나 SLAM의 핵심 기술 중 하나이다.


## 1. 좌표계 설정
가장 먼저 월드 좌표계 및 이미지 평면 좌표계를 설정한다. 적절한 좌표계를 설정해야만 이후의 연산 과정이 심플해지고 최종 연산 결과를 렌더링에 이용하기도 편리하다.

여기서는 테니스 코트 중심을 원점으로 하는 월드 좌표계를 설정하였고, 이미지 평면의 중심을 원점으로 하는 이미지 좌표계를 설정하였다.


## 2. Intrinsic Matrix 계산
카메라 모델은 Intrinsic Matrix와 Extrinsic Matrix로 구성된다. Intrinsic Matrix는 카메라의 이미지 센서와 렌즈의 하드웨어적인 파라미터에 의해 고정된 값으로 

예제에서는 아이폰 6 Plus의 후면 카메라를 사용했는데 애플은 모든 아이폰 카메라의 파라미터를 제공하므로 해당 값을 그대로 이용할 수 있다. 만일 카메라의 파라미터를 구할 수 없는 경우 OpenCV에서 제공하는 카메라 캘리브레이션 기능을 활용하면 어렵지 않게 구할 수 있다.


## 3. Extrinsic Matrix 추정
[첫번째 포스팅](https://hyun-je.github.io/vision/2019/02/07/tennis_court_line_detection_1.html)에서 구한 4개의 기준 포인트를 사용할 것이다. Extrinsic Matrix는 3x4 행렬이므로 총 12개의 미지수가 있으므로 원래는 더 OpenCV의 solvePnP 함수는 최소 4개의 좌표만 주어지면 반복적으로 오차를 줄여나가며 추정해 나간다. 초기값을 정답과 어느정도 유사하게 제공해주면 훨씬 더 좋은 결과물을 얻을 수 있다. 테니스 코트의 규격, 사람의 키, 카메라 촬영 각도 등을 고려하여 

$$2x^3$$

When \(a \ne 0\), there are two solutions to \(ax^2 + bx + c = 0\) and they are
\[x = {-b \pm \sqrt{b^2-4ac} \over 2a}.\]


## 4. Eular 각도 계산


## 5. 테니스 코트 렌더링


## 결론



## 참고자료
- [애플 개발자 문서 - iOS Device Compatibility Reference](https://developer.apple.com/library/archive/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/Cameras/Cameras.html)
- [OpenCV Documentation - Camera Calibration and 3D Reconstruction](https://docs.opencv.org/4.0.0/d9/d0c/group__calib3d.html)