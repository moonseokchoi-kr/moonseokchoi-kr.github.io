---
layout: post
title:  "[C++][WinAPI] Create Game Engine -3 Key Manager, Double Buffering"
summary: "winapi game engine"
author: moon
date: '2021-07-20 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/2
usemathjax: true
---
## 렌더링 개선하기

---

- 기존의 렌더링은 화면을 지웠다 다시 그리는 방식으로 하다보니 깜빡임 현상때문에 제대로 렌더링이 된다고 할 수 없다.
- 위의 방식은 지우고 있는 순간을 우리가 볼수도 있고, 화면이 그려진 순간을 보고 있을수도 있기 때문에 이를 해결해야한다.
- 이를 해결하기 위한 방법은 프레임 버퍼를 두개를 사용하는것이다.

    ## 이중버퍼링

    ---

    - 윈도우(창)은 화면을 그리는영역을 픽셀로 구분하는데 이 픽셀들을 비트맵이라고 한다.
    - 이러한 비트맵을 복사하여 하나더 만든다.
    - 그리고 복사한 비트맵을 가지고있는 DeviceContext를 연결해준다.
    - 이때 연결을 하는 방법은 다음과 같다.

        ```cpp
        	m_hBit = CreateCompatibleBitmap(m_hDC, m_ptResolution.x, m_ptResolution.y);
        	m_memDc = CreateCompatibleDC(m_hDC);

        	//기본값 삭제
        	HBITMAP hOldBit = (HBITMAP)SelectObject(m_memDc, m_hBit);

        	DeleteObject(hOldBit);
        ```

    - 이제 새로만든 DC에서 지우고 그리는 작업을 하고 hwnd에서 할당받은 DC은 화면의 flotting만 담당하게된다.

        ```cpp
        void CCore::render()
        {
        	Rectangle(m_memDc, -1, -1, m_ptResolution.x + 1, m_ptResolution.y + 1);
        	Vec2 vPos = g_obj.GetPos();
        	Vec2 vScale = g_obj.GetScale();
        	//그리기 작업, 화면을 덧 그리는식으로 그림(메시지와는 다른 방식)
        	Rectangle(m_memDc,
        		vPos.x - vScale.x / 2.f,
        		vPos.y - vScale.y / 2.f,
        		vPos.x + vScale.x / 2.f,
        		vPos.y + vScale.y / 2.f);
        	//이 함수를 통해 픽셀을 하나씩 복사를 하게된다.
        	BitBlt(m_hDC, 0, 0, m_ptResolution.x, m_ptResolution.y, m_memDc, 0, 0, SRCCOPY);
        }
        ```

    - 프레임의 감소하는대신 움직임이 부드러워지고 실제 게임과 같은 화면을 이제 기대할 수 있다.

## 키매니저

---

- 사용자의 입력처리를 담당하는 클래스이다.
- 매니저가 따로 필요한이유
    - 프레임의 동기화를 위해서
        - 동일 프레임 내에서 같은 키에 대해, 동일한 이벤트를 발생시키기 위해
            - 매니저가 없고 이벤트를 각각처리 할시, 어떤건 입력을 입력받고 어떤것은 못받을 수도 있다.
    - 키 입력 이벤트에 대한 구체적인 정의
        - 막 눌렀는지, 아직 누르고 있는지, 연타하고 있는지등
    - 비활성화시 키입력이 들어가지 않도록 하는 처리를 담당
- 키입력 이벤트

- 비활성화시 키입력 설정
    - 윈도우의 포커싱이 변경되었을때 현재 입력중인 키들의 상태를 변경
    - 윈도우가 비활성화 되면 모든 키의 입력이 해제된 상태로 되어야한다.
    - 탭과 HOLD에서 AWAY로 , AWAY에서 NONE으로 각각 이동시켜주고 최종적으로 모두 NONE 상태를 만든다.