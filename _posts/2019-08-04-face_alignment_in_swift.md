---
layout: post
toc: true
title: "Swift로 Face Alignment 구현"
categories: vision
tags: [swift, vision, openface]
---

Face Alignment는 얼굴 관련 알고리즘 및 머신러닝에서 필요로하는 전처리 방법으로 
표준화된 눈, 코, 입 위치가 되도록 변환하는 것을 말한다.

얼굴의 각도, 크기, 비율 등에 관계 없이 좀 더 일정한 추출하는데 도움이 된다.

OpenCV 등의 제공되나 iOS 또는 
또한 OpenCV는 CPU만 이용하여 연산되므로 성능면에서도 불리하다.

최적화된 코드를 만들어보고자 한다.

### 1. Face Landmark 추출
Apple에서 제공하는 Vision 프레임워크의 Face Landmark 추출 기능을 활용한다.
`VNDetectFaceLandmarksRequest`를 요청하면 이미지 상의 얼굴 개수 만큼 `VNFaceObservation`의 배열이 만들어진다.
각 `VNFaceObservation`에는 `VNFaceLankmarks2D` 타입의 `landmarks` 프로퍼티를 통해 각 얼굴 포인트의 좌표 값이 기록되어 있다.

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

필요한 포인트는 좌우 눈의 점과 코의 이다.
다음과 같다.

`leftEye[0]`
`rightEye[4]`
`nose[4]`

### 3. Matrix 생성

### 4. Affine Transform 수행

## 결론


## 참고자료
[애플 개발자 문서 - VNDetectFaceLandmarksRequest](https://developer.apple.com/documentation/vision/vndetectfacelandmarksrequest)
[OpenFace 프로젝트 페이지](https://cmusatyalab.github.io/openface/)
