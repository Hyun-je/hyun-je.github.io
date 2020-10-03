---
layout: post
toc: true
title: "Swift iOS 프로젝트에 OpenCV 사용"
categories: ios
tags: [ios, swift, opencv, vision]
---

OpenCV는 실시간 비전 프로세싱을 위한 라이브러리로 C++로 작성되었다. OpenCV는 잘 알려진 거의 모든 비전 알고리즘이 포함되어 있기 때문에 각 알고리즘을 일일히 구현할 필요가 없는데다 성능도 뛰어나기 때문에 비전 관련 프로그램을 만드는데 있어 필수적이라고 할 수 있다. 

OpenCV는 Java와 Python 등의 언어로는 바인딩되어 있어 편리하게 사용할 수 있지만 Swift에서 사용하기 위해서는 조금은 복잡한 셋업 과정이 필요하다. 이 과정을 기록하고 공유하기 위해 포스팅을 작성하였다.

**개발 환경**
- Xcode 12.0
- Swift 5
- OpenCV 4.4.0


### 1. OpenCV 프레임워크 다운로드
[OpenCV 공식 사이트](https://opencv.org/)의 Releases 페이지에서 최신 버전의 iOS용 프레임워크 패키지를 다운 받는다.

다운 받은 프레임워크 패키지는 OpenCV를 사용할 프로젝트의 하위 폴더에 옮겨 놓는다. (아래 예제에서는 Framworks 폴더 생성 후 그 안에 배치하였다)

![opencv_website](https://user-images.githubusercontent.com/7419790/94982534-35185900-0576-11eb-82e3-20999a4e5187.jpg)


![folder](https://user-images.githubusercontent.com/7419790/94983038-36e41b80-057a-11eb-8eb1-aa1269b43676.png)


### 2. 프로젝트 설정
Target의 **Framwork, Libraries, and Embedded Content** 목록에 OpenCV 프레임워크를 추가한다. 이 때 Embed 설정은 **Embed Without Signing**을 선택해야 한다.

![framework_setup](https://user-images.githubusercontent.com/7419790/94982801-8590b600-0578-11eb-9fa6-28557ad602e9.png)


### 3. Wrapper 클래스 제작
OpenCV는 C++로 작성되었기 때문에 Swift에서 사용하기 위해서는 Wrapper 클래스가 필요하다. Wrapper 클래스를 작성하기 위한 파일은 아래와 같이 `NSObject`의 서브클래스로 Objective-C 언어를 사용하도록 생성한다.

Wrapper 클래스는 필요한 만큼 여러개를 생성하여도 무방하므로 가급적 기능별로 분할하는 편이 좋다.

![](https://user-images.githubusercontent.com/7419790/94983219-a1e22200-057b-11eb-8a6c-e2c88b997425.png)

![](https://user-images.githubusercontent.com/7419790/94983223-a60e3f80-057b-11eb-959d-b833caef7bfc.png)

Swift 프로젝트에 Objective-C 파일을 처음 추가하는 경우 아래와 같이 Bridging Header를 생성할지 묻는 창이 뜬다. 이 때 반드시 **Create Bridging Header** 버튼을 눌러 주어야 한다.
![](https://user-images.githubusercontent.com/7419790/94983226-a73f6c80-057b-11eb-97a3-4822534de459.png)


### 4. Bridging Header 설정

생성된 Bridging Header에는 처음에 아무 내용도 없을 것이다. 여기에는 프로젝트에 만들어진 모든 Wrapper 클래스의 헤더 파일을 `#import` 하기만 하면 된다.

``` C
#import "OpenCVWrapper.h"
#import "LineDetector.h"
#import "HOG.h"
...
```