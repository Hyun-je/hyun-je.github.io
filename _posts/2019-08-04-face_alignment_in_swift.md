---
layout: post
toc: true
title: "Swift 및 iOS Framework 기반 Face Alignment 구현"
categories: vision
tags: [swift, vision, openface]
---

Face Alignment는 얼굴 관련 알고리즘 및 머신러닝에서 필요로 하는 전처리 방법으로 원본 이미지에 포함된 얼굴의 각도, 크기, 비율 등에 관계 없이 표준화된 크기 및 눈, 코, 입 위치를 갖는 얼굴 이미지가 되도록 이미지를 Affine Transfrom 및 Crop 하는 것을 말한다.

Face Alignment는 매우 널리 사용되는 전처리 방법이므로 Python과 OpenCV를 이용한 코드는 GitHub 등에서 쉽게 찾을 수 있다. 그러나 iOS 환경에서 C++로 작성된 OpenCV를 구동하기 위해서는 다소 복잡한 Bridging 과정이 필요하며 또한 CPU만 이용하여 연산되므로 성능면에서도 불리하다는 문제가 있다.

따라서 얼굴 인식 관련 iOS 앱 개발 시 유용하게 사용할 수 있는 Swift 및 Apple 디바이스에 최적화된 Face Alignment 코드를 만들어보고자 한다.


### 1. Face Landmark 추출
Apple에서 제공하는 Vision 프레임워크의 Face Landmark 추출 기능을 활용한다.

`VNDetectFaceLandmarksRequest`를 생성 후 실행하면 이미지에 포함된 얼굴 개수 만큼의 `VNFaceObservation`의 배열이 만들어진다. 각 `VNFaceObservation`에는 `VNFaceLankmarks2D` 타입의 `landmarks` 프로퍼티를 통해 각 얼굴 포인트의 좌표 값이 기록되어 있다.

![face_landmarks](https://user-images.githubusercontent.com/7419790/94780635-bd2a2180-0403-11eb-9ddf-ef3c78f406ae.jpg)


### 2. Lankmark 선택
`VNFaceLankmarks2D` 타입은 아래와 같은 프로퍼티를 가지며 이 프로퍼티를 통해 전체 포인트 또는 특정 얼굴 요소의 포인트를 얻을 수 있다.

- 모든 포인트 : `allPoints`
- 얼굴 중심축 : `medianLine`
- 얼굴 윤곽 : `faceContour` 
- 눈 : `leftEye` `rightEye`
- 눈동자 : `leftPupil` `rightPupul`
- 눈썹 : `leftEyebrow` `rightEyebrow`
- 코 : `nose` `noseCrest`
- 입술 : `OuterLips` `innerLips` 

이 중에서 `OUTER EYES AND NOSE` 기준 정렬을 수행하기 위해 필요한 요소는 아래와 같다. 그리고 각 요소에 대해 다음과 같이 이미지 평면상의 x, y 좌표를 정의하였다.

- `leftEye[0]` → `(x1, y1)`
- `rightEye[4]` → `(x2, y2)`
- `nose[4]` → `(x3, y3)`

![face_landmarks_selection](https://user-images.githubusercontent.com/7419790/94782434-2a3eb680-0406-11eb-94be-29a207558b4f.jpg)


### 3. 좌표 변환
Vision 프레임워크와 CoreImage에서 사용하는 좌표계는 
정확한 Transform Matrix를 구하기 위해서는 고려하여 연산을 수행하여야 한다.

### 4. Matrix 생성
역행렬 연산 및 행렬 곱 연산은 Accelerate 프레임워크를 이용하여 성능을 최적화 한다.

### 5. Affine Transform 수행

## 결론
https://github.com/Hyun-je/Openface_Face_Embedding-swift

## 참고자료
[애플 개발자 문서 - VNDetectFaceLandmarksRequest](https://developer.apple.com/documentation/vision/vndetectfacelandmarksrequest)
[OpenFace 프로젝트 페이지](https://cmusatyalab.github.io/openface/)
