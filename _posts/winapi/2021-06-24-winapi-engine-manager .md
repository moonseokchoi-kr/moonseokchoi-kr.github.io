---
layout: post
title:  "[C++][WinAPI] Create Game Engine -2 Manager Class"
summary: "winapi game engine"
author: moon
date: '2021-06-23 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/manager
usemathjax: true
---
- 게임제작 1에서 이어지는 내용

    [WinApi[게임엔진 제작-1]](https://www.notion.so/WinApi-1-d14b8b0e38da43cba2bf003c88fbac68)

## 게임창 크기 설정하기

---

- 입력받은 해상도로 창크기를 변경

    ```cpp
    SetWindowPos()// 이 함수를 이용한다.
    ```

- 이 함수로 변경되는 함수는 타이틀바, 메뉴바등의 화면의 모든 구성요소를 포함하고있는 크기를 말한다.(Win7, Win8는 상단바가 두꺼운편이고, Win10는 상단바가 거의 없다.)
- 윈도우 버전마다 크기가 다르기 때문에 자신이 원하는 해상도를 얻는 함수가 있다.

    ```cpp
    AdjustWindowRect();// 사용자가 원하는 해상도 크키를 상단바를 포함하여 계산
    ```

    - RECT를 자료구조로 받기 때문에 따로 선언이 필요
    - 2번째 인자는 어떠한 방식으로 윈도우를 설정할지 선택(메뉴바가 필요할지, 최소화버튼이 있어야할지 등등..)
    - 마지막은 메뉴바를 포함한 크기로 계산할지 묻는다
    - 결과는 처음 설정한 RECT자료구조에 들어간다.
- 이 두함수를 이용하여 창을 설정한다.

    ```cpp
    int CCore::init(HWND _hWnd, POINT _ptResolution)
    {
    	m_hWnd = _hWnd;
    	m_ptResolution = _ptResolution;

    	// 해상도에 맞게 윈도우 크기 조정
    	RECT rt = {0,0,m_ptResolution.x, m_ptResolution.y};
    	//리턴값이 너무 클때 주소형태로 받아오도록설정
    	AdjustWindowRect(&rt, WS_OVERLAPPEDWINDOW, true);
    	//결과값을 보면 top,left의 음수의 값이 들어간다. 이 의미는 메뉴를 구분하는 선이 차지할 픽셀을 말하는것이다.
    	//따라서 둘을 더해야 실제 크기가 나온다
    	SetWindowPos(_hWnd, nullptr, 1920, 1080, rt.right - rt.left, rt.bottom - rt.top, 0);

    	return S_OK;
    }
    ```

## 그래픽 버퍼를 만드는 과정

---

- 메세지처리방식을 이용하지 않는다. 게임에서는 매우 비효율적이다.
    - 메세지처리방식은 무효화영역이 발생되어야 호출된다는 문제점이 있다.
    - 변화를 강제적으로 발생시켜야한다는 문제가 있다.
- 따라서 메시지 기반이 아닌 메시지를 처리하지않는 시간에 그림을 그릴예정이다.
- 일단 그림을 그리기위해서는 Device Context(DC)를 얻어와야한다.

    ```cpp
    GetDC(HWND);
    ```

    - HDC 타입을 반환한다.
- 사용후 DC를 해제하는 함수도 있다. 소멸자에 작성해주자

    ```cpp
    Release(HWND,HDC);
    ```

- 이제 DC를 얻어서 초기화를 한다.

    ```cpp
    m_hDC = GetDC(m_hWnd);
    ```

- 이제 그림을 그리고 싶은 오브젝트에 이 DC를 넣어주면된다.

    ```cpp
    Rectancle(m_hdc,10,10,100,100)
    ```

- 이렇게 만든 렌더링의 문제점
    - 컴퓨터 사양에 따라 그려지는 속도가 다르다.
    - 전에 그려진 그림이 지워지지 않는다.

    ## 1차 개선

    ---

    - 좌표를 실수로 바꾸어 만든다.
    - 이를 위해 이전에 사용했던 POINT 구조체를 버리고 Vec2 2차원 벡터 구조체를 새로만든다.

    ```cpp
    struct Vec2
    {
    	float x;
    	float y;

    public:
    	Vec2()
    		:x(0.f),
    		 y(0.f)
    	{}
    	Vec2(float _x, float _y)
    		:x(_x),
    		y(_y)
    	{}
    	Vec2(int _x, int _y)
    		:x(float(_x))
    		,y(float(_y))
    	{}
    };
    ```

    - 이런식으로 개선하고 실행해보면 아까보다 나아졌지만, 잔상과 컴퓨터 사양에 따라 그려지는 속도가 다르다는 문제가 해결되지는 않는다.

    ## 2차 개선

    ---

    - 시간동기화를 통한 컴퓨터마다 재생속도가 다른걸 해결
    - 예를 들어 1초에 100이라는 이동거리를 이동하는데 어떤 컴퓨터는 100프레임이 나오고 어떤 컴퓨터는 200프레임이 나온다 했을때 둘이 같은 시간이 걸리려면 각각 움직여야할 거리는 얼마일까?
        - 100프레임은 1프레임, 200프레임은 0.5프레임
    - 이 질문이 시사하는것은 프레임을 역수로 치한하면 한프레임당 실제 걸리는 시간을 알 수 있다는것이다.
    - 따라서 시간을 관리하기 위해서는 타이머 클래스가 FPS(초당 프레임)의 수치를 알고, 1프레임당 시간(delta time)을 구하기만 하면 시간동기화가 가능하다.
    - 컴퓨터의 연산량은 상상을 초월하기 때문에 시간을 측청하기위해서는 매우세밀한 측정함수를 써야한다.

        ```cpp
        QueryPerformanceCounter(&m_llCurCount);//퍼포먼스 횟수 계산
        QueryPerformanceFrequency(&m_llFrequency);//빈도 계산
        ```

    - 이제 이 두함수를 이용해서 Time Manager를 만들어보자.