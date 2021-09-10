---
layout: post
title:  "[C++][WinAPI] Create Game Engine -9 Animation"
summary: "winapi game engine"
author: moon
date: '2021-07-29 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/animation
usemathjax: true
---
## 애니메이션

---

- Collider에이은 두번째 Component로 필요시에 호출 하는 함수이다.
- 애니메이션은 프레임을 기준으로 하는 프레임애니메이션이 있다.
- 프레임 애니메이션은 연속적 이미지를 빠르게 재생하여 마치 플레이어가 걷는것 과 같은 느낌을 주는 방식이다.

## Animator

---

- 위에서 설명한 애니메이션을 관리해주는 관리자 클래스
- Animator가 가져야할 것
    - 애니메이터의 소유주
    - 애니메이션을 저장하는 컨테이너(Map 키: 애니메이션 이름 )
- Animator의 기능
    - 애니메이션의 생성
    - 애니메이션 찾기
    - 재생
    - 보유중인 애니메이션 삭제
    - 애니메이션 업데이트
    - 애니메이션 렌더링

## Create Animation

---

- 프레임 애니메이션은 스프라이트가 이미지형태로 제공된다.
- 스프라이트 하나의 크기를 계산한뒤 각 좌표를 제공하여 해당 이미지를 가져올수 있도록 한다.
- 함수 내부 순서
    - 같은 이름의 애니메이션이 있는지 검사, 있으면 애니메이션을 반환
    - 없다면 새 애니메이션 객체를 만든다.
    - 현재 애니메이터와 포인터를 이용해 연결하고, 애니메이션의 Create함수를 호출한다.
    - 이렇게 생성된 애니메이션을 애니메이션 저장공간에 저장한다.

### Animation class Create 함수

---

- 이미지정보를 받아 실질적으로 애니메이션을 생성하는 함수
- 각 프레임에 대한 정보를 저장하는 구조체를 이용하여 추후 필요할때 데이터를 재생한다.

    ```cpp
    struct tFrameInfo
    {
    	Vec2 vLT, // 시작위치
    	Vec2 vSlice, //각 프레임의 크기
    	float fDuration //재생시간(속도)
    }
    ```

- 이렇게 생성된 구조체에 받아온 함수들을 이용해 각 프레임의 위치정보를 저장한다.

    ```cpp
    for (int i = 0; i < _iFrameCount; ++i)
    	{
    		frm.fDuration = _fDuration;
    		frm.vLT = _vLT+_vStep*i;
    		frm.vSlice = _vSlicePoint;

    		m_vecFrm.push_back(frm);
    	}
    ```

- 이렇게 정보를 저장했으므로 렌더링시 이 정보를 렌더링 하면 된다.

### CPlayer 구조 변경

---

- 애니메이션 클래스를 통해 외형을 렌더링하게 됨으로, CPlayer가 직접 Transparenblt을 호출할 이유가 없다.
- 함수 호출 순서
    - Scene.render()→CPlayer.render()→CAnimator.render()→CAnimation.render()
- 따라서 기존의 가지고 있던 texture멤버와 그와 관련된 코드를 지워준다

### Animation Render

---

- 애니메이션을 렌더링하기 위해서는 플레이어의 현재위치를 알아야한다. 그래야만 그 위치를 기준으로 플레이어를 그릴 수 있다.
- 따라서 애니메이터에서 오브젝트를 가져갈 수 있도록 설정해준다(GetObject함수)
- 그 이후 TransparentBlt로 출력해준다

## Animation Update

---

- 누적시간에 따라 현재 프레임을 증가시키고, 누적시간은 (누적시간) - (재생시간)의 결과로 만든다
- 만약 벡터크기만큼 프레임이 도달했다면 현재 프레임을 0으로 만든다.

- 현재 업데이트는 무한 반복이기 때문에 무한반복을 없애주는 코드도 있어야한다.
- 무한반복 없애기
    - 애니메이터에 반복을 체크하는 bool값과 애니메이션에 재생완료를 bool값을 주고 둘사이의 논리체크로 재생을 시키면된다.
    - 이 검사를 할때 애니메이터의 bool값을 먼저 체크하는게 좋다, C++ 언어 최적화로 AND체크에서 첫값이 bool이면 뒤에값의 검사를 무시하기 때문이다.

## 애니메이션 구조체 수정

---

- 대표적으로 탑뷰형식의 게임을 만들때 지금의  위치지정방식에 문제가 있다.
    - 현재는 물체의 중심을 기준으로 위치를 잡기 때문에 문제가 있다.
    - 예를 들어 바닥에 오브젝트를 설치한다 했을때 위치를 기준으로 바닥에 설치할텐데 이때 물체의 중심이 기준이 되기때문에 그만큼 빼줘야하는 예외처리를 해줘야한다.
    - 이와 같은 문제를 없애기 위해 바닥을 기준으로 보통 물체의 위치를 잡는다.
    - 이미지의 중심을 기준으로 모든 작업이 진행되기 때문에 탑뷰에서 보면 같은 라인이 아니더라도 충돌 판정이 일어난다.
    - 따라서 충돌체 처럼 애니메이션에서 오프셋을 주어서 출력과 충돌이 자연스럽게 이루어지도록 한다.