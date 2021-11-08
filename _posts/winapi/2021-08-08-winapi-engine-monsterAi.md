---
layout: post
title:  "[C++][WinAPI] Create Game Engine -17 Monster AI"
summary: "winapi game engine"
author: moon
date: '2021-08-03 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/ai
usemathjax: true
---
- FSM(Finite State Machine)
    - 유한한 상태를 가진 머신을 말하는것으로 프로그래머가 설정한 상태값에 맞게 동작을 한다.
    - SceneManager나 Animation이 이러한 모델로 구현되어있다.
    - 몬스터의 다양한 행동의 기반은 이 State Machine의 패턴의 따라 나오게된다.

- FSM제작
    - FSM은 State를 설정하고 생성하는 곳이 아닌 기존의 있는 State들에 전환을 담당한다. 그렇기 때문에 State를 위한 클래스를 생성한다.
    - Enum값으로 스테이트의 이름을 관리하게 되면 파생이나 확장이 매우편해진다. 그렇기 때문에 몬스터 상태를 결정할 enum을 설정한다.
    - State의 경우 기본생성자를 지우고, 무조건 State 열거값을 받도록한다.
        - State의 종류를 무조건 알아야하기때문에 자식State를 초기화할때 입력하도록한다.
    - 상태에 진입하고 탈출할때 해야하는 일들이 있기 때문에, 부모클래스에서는 이부분을 순수가상함수로 구현해, 자식들에서 구현할수 있도록한다.
    - 이제 업데이트를 호출하게되는데, 기존의 Monster 클래스에서 했던 업데이트를 이제 AI가 담당하게되고,
    - AI는 State의 업데이트를 호출해 상태에 맞게 호출된 상태클래스의 업데이트가 시작된다.

    
- FSM사용
    - function.h에 상태를 생성하는 함수를 미리 정의 해두면 앞으로 여러몬스터를 작성할때 반복적인 코드를 적는 작업을 줄일수 있다.
        - 이러한 패턴을 Factory Pattern이라고 한다.
        - Factory클래스의 생성자를 없애고, 정적멤버함수를 통해 몬스터를 생성해내는 공장을 만든다.
    - 상태값에는 결국 한계가 있기 때문에 사람이 아닌 컴퓨터가 조종한다는 느낌을 지울수가 없기 때문에 유한한 상태머신이라고 불리는 이유이다.