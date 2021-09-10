---
layout: post
title:  "[C++][WinAPI] Create Game Engine -12 Unity Build"
summary: "winapi game engine"
author: moon
date: '2021-08-03 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/unitybuild
usemathjax: true
---
- 현재 만들어진 소스들을 빌드하는 속도가 확 느려진 상태이기때문에 한번 빌드를 최적화 할 필요가 있다.
    - 속도가 느려진 이유는 분할구현으로 서로가 서로를 링크하는식으로 연결되다 보니 컴파일러 의 확인 작업이 배로 늘어났기 때문이다.
- 팀 작업이라면 각자의 컴퓨터를 연결하여 빌드의 대한 부담을 나눠가지는 방식이 있으나, 개인에서는 불가능하다.
- 그래서 개인이 할수 있는 극적인 빌드 속도 향상은 Unity Bulid방식이다.
    - 기존의 분할구현되어있던 파일을 빌드할때만 하나의 파일을 합치는 방식으로 빌드하는것
    - 서로간의 링크관계의 복잡성이 확 줄어 빌드의 속도가 상승한다.

## 유니티 빌드 순서

---

- 프로젝트 속성에서 모든 CPP파일을 빌드 제외시킨다.
- 임시로 CPP를 생성하여 만들고 거기에 CPP파일을 전부 include시킨다.
- 다만, 하나의 CPP파일에 적을수 있는 글자 수 제한이 있기 때문에 본인이 잘 판단해서 분할을 해야한다.
- 과거에는 직접 이런 프로그램을 만들었지만, 지금은 정식기능으로 들어왔다(아직 베타버전)

## VisualStudio에서 적용

---

- 기능이 숨겨져 있기때문에 직접 속성파일을 수정해야한다.

    ```cpp
    <PropertyGroup>
    	<EnableUnityGroup>true</EnableUnityGroup>
    </PropertyGroup>
    ```

- 위 xml텍스트를 프로퍼티 그룹 사이 아무곳에나 추가하고 저장한다.
- 그후 속성창에서  다중프로세스 빌드와, 미리컴파일된 헤더를 사용으로 변경하고 저장위치를 현재 프로젝트 디렉토리로 지정한다. 미리컴파일된 헤더 기능은 아예 꺼버린다.(문제가 있는데 해결해야 쓸수 있을듯)