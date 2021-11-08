---
layout: post
title:  "[C++][WinAPI] Object Control"
summary: "winapi programming"
author: moon
date: '2021-06-22 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic
permalink: /blog/cpp/winapi/basic
usemathjax: true
---
## 오브젝트 움직이기

---

- windows에선 POINT 구조체를 제공해 좌표를 지정할 수 있도록 한다.

    ```cpp
    POINT gObjectPoint = {500,300}
    ```

- 이를 이용해 사각형의 위치를 동적으로 만들수 있다.

    ```cpp
    Rectancle(hdc, gObjectPoint.x-gObjectScale.x/2,gObjectPoint.y-gObjectScale.y/2,
    gObjectPoint.x+gObjectScale.x/2, gObjectPoint.y+gObjectScale.x/2)
    ```

- 이후 키다운 이벤트에서 이를 처리해주면 된다.
    - 이때 무효화영역을 강제로 활성화시켜줘야 우리가 원하는 결과를 얻을수 있다.

## 마우스를 이용해 원하는 위치에 사각형 생성

---

- 먼저 사격형의 크기와 위치를 기억할 구조체를 만든다.

    ```cpp
    struct t_objPoint
    {
        POINT g_ptObjPos;
        POINT g_ptObjScale;
    };

    //좌상단
    POINT g_ptLT = { 400, 300};
    //우하단
    POINT g_ptRB = { 100, 100 };
    ```

- 사각형의 위치를 저장한 자료구조를 만들고 클릭이벤트를 위한 boolean 값을 선언한다.

    ```cpp
     std::vector<t_objPoint> t_objPoints;
    bool isClick = false;
    ```

- 다음과 같이 그리는 이벤트를 설정한다.

    ```cpp
    if (isClick)
    {
         Rectangle(hdc, g_ptLT.x, g_ptLT.y, g_ptRB.x, g_ptRB.y);
    }
                

     for (int i = 0; i < t_objPoints.size(); ++i)
     {
    		 Rectangle(hdc,
          t_objPoints[i].g_ptObjPos.x - t_objPoints[i].g_ptObjScale.x / 2,
           t_objPoints[i].g_ptObjPos.y - t_objPoints[i].g_ptObjScale.y / 2,
          t_objPoints[i].g_ptObjPos.x + t_objPoints[i].g_ptObjScale.x / 2,
          t_objPoints[i].g_ptObjPos.x + t_objPoints[i].g_ptObjScale.y / 2
          );
     }
    ```

- 마우스를 이용한 이벤트를 다음과 같이 설정한다

    ```cpp
    //마우스 클릭시 그리는것 시작	
    case WM_LBUTTONDOWN:
    	{
        isClick = true;
    		g_ptLT.x = LOWORD(lParam);
    		g_ptLT.y = HIWORD(lParam);
    		InvalidateRect(hWnd, nullptr, true);

    	}
    	break;
    //마우스 그림그리기
    	case WM_MOUSEMOVE:
    	{
    		g_ptRB.x = LOWORD(lParam);
    		g_ptRB.y = HIWORD(lParam);
    		InvalidateRect(hWnd, nullptr, true);
           
    	}
            break;
    //버튼을 놓으면 벡터에 사각형 추가
    	case WM_LBUTTONUP:
    	{
            t_objPoint info{};

            info.g_ptObjPos.x = (g_ptLT.x + g_ptRB.x) / 2;
            info.g_ptObjPos.y = (g_ptRB.y + g_ptLT.y) / 2;

            info.g_ptObjScale.x = abs(g_ptLT.x - g_ptRB.x);
            info.g_ptObjScale.y = abs(g_ptLT.y - g_ptRB.y);

            t_objPoints.push_back(info);
            isClick = false;
            InvalidateRect(hWnd, nullptr, true);
    	}
        break;
    ```

- 왜 화면이 깜빡이나요?
    - 화면을 재생시키는 타이밍 때문이다. 화면을 그리는 버퍼가 하나 뿐이기 때문에 화면을 지우고 다시 그리는데, 화면의 송출과 화면이 그리는 타이밍이 맞지 않기때문에 지웠던 화면에서 갑자기 다 그려진 화면이 나오기 때문에 화면이 깜빡이는것처럼 보인다.
    - 이를 해결하기 위해 프레임 버퍼를 두개를 이용해 스왑하는 방식으로 렌더링을 하면 해결 할 수 있다.(이게 스왑체인이라는 기능이다)

## 메세지 기반 방식에서 탈출하기

---

- GetMessage 함수를 게임에서 이용할때의 문제점
    - 메세지가 없어도 종료되지 않고 대기한다.
    - 메세지가 있어야만 루프가 실행된다.
        - WM_TIME이벤트를 강제로 일으키는 방법을 통해 진행할 수 있지만 그럴경우 매우 실행속도가 느려진다.
- PeekMessage의 이용
    - 메세지가 있던 없던 결과 값을 반환
    - 이를 통해 원래 대기하며 시간을 보낸부분에 게임루프를 사용할 수 있게되었음

    ```cpp
    while (true)
    	{
    		if (PeekMessage(&msg, nullptr, 0, 0, PM_REMOVE))
    		{
    			if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
    			{
    				TranslateMessage(&msg);
    				DispatchMessage(&msg);
    			}
    		}
    		else
    		{
    			//대부분의 게임루프를 돌리는곳
    		}

    	}
    ```