---
layout: post
toc: true
title: "SceneDelegate 이해하기"
categories: ios
tags: [ios, swift, storyboard, ui]
---

Xcode 11.0 부터 iOS 프로젝트를 생성하는 경우 변경점이 하나 생겼다. Xcode에서 자동으로 생성해주는 파일에 SceneDelegate라는 파일이 추가된 것이다.

Scene이라는 단어가 포함되어 있기 때문에 화면에 보여지는 무언가와 관련된 것임을 짐작할 수 있는데 과연 SceneDelegate는 무엇인지, 어떻게 사용해야 하는지 알아보았다.


## iOS 13의 새로운 기능
iOS/iPadOS 13에서는 새로 생긴 기능 중 하나가 Multiple Windows 기능이다. Multiple Windows 기능은 1개의 앱 프로세스가 여러개의 윈도우를 가질 수 있도록 하는 기능이다. 이를 활용하면 두개의 사파리 앱을 띄워놓고 동시에 2개의 웹페이지를 브라우징 하는 등의 작업이 가능해지게 된다.

Multiple Windows 기능은 기본적으로는 iOS/iPadOS의 생산성을 높이기 위한 목적이라고 볼 수 있으나 장기적인 관점에서는 MacOS 앱과의 간극을 줄이고 Catalyst를 통한 통합을 용이하도록 만드는 과정으로 보여진다.

그런데 1개의 프로세스에 1개의 윈도우만 가지도록 제약이 있었던 시절에 만들어진 아키텍처로는 여러개의 윈도우의 Lifecycle을 관리하기에 어려웠고 이를 개선 하기 위해 SceneDelegate를 도입하게 되었다.


## AppDelegate와 SceneDelegate
SceneDelegate를 이해하기 위해서는 AppDelegate의 역할에 대해서 알고 있어야 한다.

원래 AppDelegate는 앱의 Launch, Terminate와 같은 Process Lifecycle과 Foreground, Background 상태와 같은 UI Lifecycle 두가지를 모두 관리하였다. 각각의 Scene에 대해서 필요해지게 된다.

AppDelegate에서 UI Lifecycle 관리 부분만 따로 떼어내 만든 것이 SceneDelegate이다.


## SceneDelegate를 사용하지 않는 방법
Deployment Target을 iOS 13 이전으로 설정하는 경우 SceneDelegate를 사용할 수 없다. 또 코드를 유지하고자 하는 경우 SceneDelegate를 사용하지 않는 방법이 있다.

SceneDelegate는 기본적으로 info.plist에서 정적으로 할당된다.




## 참고자료
[WWDC19 - Architecting Your App for Multiple Windows](https://wwdc.io/share/wwdc19/258)
