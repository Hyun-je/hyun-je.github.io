---
layout: post
toc: true
title: "SceneDelegate 이해하기"
categories: ios
tags: [ios, swift, ui, wwdc19]
---

Xcode 11.0 부터 iOS 프로젝트를 생성하는 경우 변경점이 하나 생겼다. Xcode에서 자동으로 생성해주는 파일에 `SceneDelegate.swift`라는 파일이 추가된 것이다. Scene이라는 단어가 포함되어 있기 때문에 화면에 보여지는 무언가와 관련된 것임은 짐작할 수 있는데 과연 `SceneDelegate`는 무엇인지, 어떻게 사용해야 하는지 알아보았다.


## iOS 13의 새로운 기능
iOS/iPadOS 13에서는 새로 생긴 기능 중 하나가 Multiple Windows 기능이다. Multiple Windows 기능은 1개의 앱 프로세스가 여러개의 윈도우를 가질 수 있도록 하는 기능이다. 이를 활용하면 두개의 사파리 앱을 띄워놓고 동시에 2개의 웹페이지를 브라우징 하는 등의 작업이 가능해지게 된다.

![multiple_windows](https://user-images.githubusercontent.com/7419790/94760917-bc32c900-03de-11eb-9912-46a9e30f8a83.jpg)

>  Multiple Windows 기능은 기본적으로 iOS/iPadOS의 생산성을 높이기 위한 목적이라고 볼 수 있으나 전략적인 측면에서는 MacOS 앱과의 간극을 줄이고 Catalyst를 통한 통합을 용이하도록 만드는 과정으로 보여진다.

애플은 Multiple Windows 기능을 구현하는데 좀 더 합리적이고 유연한 아키텍처를 제시하기 위해 SceneDelegate와 이와 관련된 몇 가지 새로운 개념을 도입하게 되었다.


## AppDelegate와 SceneDelegate
`SceneDelegate`를 이해하기 위해서는 `AppDelegate`의 역할에 대해서 알고 있어야 한다.

원래 `AppDelegate`는 앱의 Launch, Terminate와 같은 Process Lifecycle과 Foreground, Background 상태와 같은 UI Lifecycle 두가지를 모두 관리할 수 있도록 만들어졌다. 1개의 앱 프로세스가 1개의 UI 인스턴스만 가지던 과거에는 이러한 구성에 아무런 불합리한 점이 없었다.

그런데 iOS 13에서부터 Multiple Windows 기능이 추가되면서 1개의 앱 프로세스가 여러개의 UI 인스턴스를 가지게 되었다. 
서로 다른 인스턴스의 개수를 갖는 프로세스와 UI 관리가 `AppDelegate`에 결합된 형태는 더 이상 유지할 수 없게 되었다. 결과적으로 `AppDelegate`에서 UI Lifecycle 관리 부분만 따로 떼어내 만든 것이 `SceneDelegate`이다.

![wwdc_screenshot](https://user-images.githubusercontent.com/7419790/94777073-cb753f00-03fd-11eb-81ec-6dd635b5cecd.png)
![wwdc19_screenshot](https://user-images.githubusercontent.com/7419790/94761132-411de280-03df-11eb-8386-c567ad92eca0.png)


## SceneDelegate를 사용하지 않는 방법
Deployment Target을 iOS 13 이전으로 설정하는 경우 `SceneDelegate`를 사용할 수 없다. 또 기존의 방식대로 AppDelegate에서 프로세스와 처리하도록 구성된 기존 코드를 유지하고자 하는 경우 `SceneDelegate`를 사용하지 않는 방법이 있다.

1. `SceneDelegate.swift` 파일 삭제
2. `info.plist`에서 `Application Scene Manifest` 키 삭제
3. `AppDelegate`에 `UIWindow` 정의
``` swift
var window: UIWindow?
```
4. `AppDelegate`의 `UISceneSession` 관련 함수 삭제
``` swift
// func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration
// func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>)
```

## 참고자료
- [WWDC19 - Architecting Your App for Multiple Windows](https://wwdc.io/share/wwdc19/258)
- [WWDC19 - Introducing Multiple Windows on iPad](https://developer.apple.com/videos/play/wwdc2019/212/)
- [애플 개발자 문서 - UIScene](https://developer.apple.com/documentation/uikit/uiscene)
- [Raywenderlich - Adopting Scenes in iPadOS](https://www.raywenderlich.com/5814609-adopting-scenes-in-ipados)
