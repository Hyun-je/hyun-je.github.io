---
layout: post
toc: true
title: "OpenCV 테니스 코트 인식 1 (라인 검출)"
categories: vision
tags: [opencv, python, vision]
---


OpenCV를 이용해 이미지에서 테니스 코트 라인 추출을 하는 과정을 설명합니다.


처리 과정


이미지 처리 및 테니스 코트 라인 검출은 아래과 같은 순서로 이루어집니다. 최종적으로 테니스 코트 상의 4개의 포인트의 이미지 평면 상의 좌표가 검출 되며 이 좌표는 이후의 Camera Pose 연산을 위한 활용됩니다.

## 1. 이미지 전처리

주변 환경과 카메라의 밝기, 명암비 등에 의한 인식률의 영향을 줄이기 위해 적절한 전처리 과정을 적용합니다. 아래의 OpenCV의 

- Color Space 변경 (RGB → 그레이스케일)
- Histogram 이퀄라이징 : 이미지의 명암비를 일정하게 하여
- Gaussian Blur : 노이즈의 영향
- Morphology 연산 : 흰 코트 라인이 강조되도록 합니다

``` python
 // 이미지 컬러스페이스 변환
  cv::Mat img_GRAY;
  cv::cvtColor(img_RGBA , img_GRAY , cv::COLOR_RGBA2GRAY);

  // 히스토그램 이퀄라이징
  cv::Ptr<cv::CLAHE> clahe = cv::createCLAHE();
  clahe->setClipLimit(2.0);
  clahe->setTilesGridSize(cv::Size(8, 8));
  clahe->apply(img_GRAY, img_GRAY);


  // 가우시안 블러 적용
  cv::GaussianBlur(img_GRAY, img_GRAY, cv::Size(5,5), 0);

  // 모폴로지 적용
  cv::morphologyEx(img_GRAY, img_GRAY, cv::MORPH_CLOSE, cv::Mat::ones(5, 5, CV_8U));
```


## 2. Edge 검출

강조되
``` python
    // Canny Edge 적용
    cv::Mat img_Edge;
    cv::Canny(img_GRAY, img_Edge, 50, 200);
```


## 3. 직선 검출
이미지에 포함된 직선 들을 검출합니다. 검출된 직선은 y절편과 기울기 값으로 구해지므로 이후에는 
``` python
    // HoughLine Transforn 수행
    std::vector<cv::Vec2f> lines;
    cv::HoughLines(img_Edge, lines, 1.0, 3.141592/180/5, 300);
```


## 4. 클러스터링


## 5. 테니스 코트 라인 검출


## 6. 코트 라인의 교점 연산

