---
layout: post
toc: true
title: "정적인 TableView 활용하기"
categories: ios
tags: [ios, swift, storyboard, ui]
---

TableView는 다수의 데이터 목록을 표시할 수 있는 효과적인 UI 요소이다. 일반적으로 데이터 목록의 개수와 내용은 런타임에서 동적으로 변경되므로 TableView 또한 delegation 패턴을 활용하여 동적으로 구성되는 것이 보통이다.

그런데 설정 메뉴나 와 같이 정적인 화면에 TableView를 활용하는 경우도 적지 않다. 이 경우 TableView의 동적인 동작방식이 불필요하게 많은 Cell Prototype 생성과 코드 작성이 필요한 걸림돌이다.

이 때 정적인 TableView를 잘 활용한다면 Interface Builder만을 활용하여 원하는 형태의 TableView를 만들 수 있다.


## 정적인 TableView의 특징
정적인 TableView는 Interface Builder 상에서 Content 속성이 Static Cells로 지정된 TableView를 의미한다. 기본적으로는 UITableViewController의 TableView의 경우에만 Static Cells 지정이 가능하며


## 정적인 TableView 활용


### 1. Static TableView 생성


### 2. 설정 메뉴 구성


### 3. Cell 높이 자동 조절 적용



## 결론


## 참고자료
