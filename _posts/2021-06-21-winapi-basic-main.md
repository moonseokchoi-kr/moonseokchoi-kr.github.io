---
layout: post
title:  "[C++][WinAPI] wMain 함수 해석"
summary: "object oriented programming"
author: moon
date: '2021-06-21 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic
permalink: /blog/data-structure/graph
usemathjax: true
---
```cpp
#define MAX_LOADSTRING 100

// 전역 변수:
HINSTANCE hInst;                                // 현재 인스턴스입니다.
WCHAR szTitle[MAX_LOADSTRING];                  // 제목 표시줄 텍스트입니다.
WCHAR szWindowClass[MAX_LOADSTRING];            // 기본 창 클래스 이름입니다.
```

- WCHAR
    - wchar_t의 이름을 재정의한것

```cpp
// 이 코드 모듈에 포함된 함수의 선언을 전달합니다:
ATOM                MyRegisterClass(HINSTANCE hInstance);
BOOL                InitInstance(HINSTANCE, int);
LRESULT CALLBACK    WndProc(HWND, UINT, WPARAM, LPARAM);
INT_PTR CALLBACK    About(HWND, UINT, WPARAM, LPARAM);

int APIENTRY wWinMain(_In_ HINSTANCE hInstance,
                     _In_opt_ HINSTANCE hPrevInstance,
                     _In_ LPWSTR    lpCmdLine,
                     _In_ int       nCmdShow)
{
    UNREFERENCED_PARAMETER(hPrevInstance);
    UNREFERENCED_PARAMETER(lpCmdLine);

    // TODO: 여기에 코드를 입력합니다.

    // 전역 문자열을 초기화합니다.
    LoadStringW(hInstance, IDS_APP_TITLE, szTitle, MAX_LOADSTRING);
    LoadStringW(hInstance, IDC_CLIENT, szWindowClass, MAX_LOADSTRING);
    MyRegisterClass(hInstance);

    // 애플리케이션 초기화를 수행합니다:
    if (!InitInstance (hInstance, nCmdShow))
    {
        return FALSE;
    }
}
BOOL InitInstance(HINSTANCE hInstance, int nCmdShow)
{
   hInst = hInstance; // 인스턴스 핸들을 전역 변수에 저장합니다.

   HWND hWnd = CreateWindowW(szWindowClass, szTitle, WS_OVERLAPPEDWINDOW,
      CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, nullptr, nullptr, hInstance, nullptr);

   if (!hWnd)
   {
      return FALSE;
   }

   ShowWindow(hWnd, nCmdShow);
   UpdateWindow(hWnd);

   return TRUE;
}

```

- hInstance
    - 실행된 프로그램의 메모리주소를 의미한다.
    - 새로운 창을 생성하기 위한 정보를 입력받는 구조체
    - 메인함수에서 지역변수로 입력받아, 초기화 함수를 거쳐 초기화가 된다.
    - hPrevInstance는 과거 windows를 위해 필요한 파라미터이나, 지금은 필요가 없다.
- IpCmdLine
    - wchar_t pointer (LPWSTR)타입으로 명령창을 이용해 문자열을 표시할 경우 사용
- LoadStringW
    - 사용할 리소스를 테이블에서 가져와 문자열을 초기화하는 함수
    - 2번째 파라미터로 가져올 리소스의 ID를 전달받는다.(resource.h에 전처리기로 설정 되어있다.)
- MyRegisterClass
    - 윈도우(프로그램 창)을 설정하는 함수
    - 내부에서 윈도우를 설정하는 구조체를 작성

    ```cpp
    ATOM MyRegisterClass(HINSTANCE hInstance)
    {
        WNDCLASSEXW wcex;

        wcex.cbSize = sizeof(WNDCLASSEX);

        wcex.style          = CS_HREDRAW | CS_VREDRAW;
        wcex.lpfnWndProc    = WndProc;
        wcex.cbClsExtra     = 0;
        wcex.cbWndExtra     = 0;
        wcex.hInstance      = hInstance;
        wcex.hIcon          = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_CLIENT));
        wcex.hCursor        = LoadCursor(nullptr, IDC_ARROW);
        wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);
        wcex.lpszMenuName   = MAKEINTRESOURCEW(IDC_CLIENT);
        wcex.lpszClassName  = szWindowClass;
        wcex.hIconSm        = LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));

        return RegisterClassExW(&wcex);
    }
    ```

- InitInstance
    - 윈도우를 생성하는함수 앞써 생성한 창의 정보를 기반으로 윈도우를 생성
    - 창은 프로세스이나, 모든 프로세스가 창은 아니다(백그라운드에서 실행되는 프로세스도 있다.)
- 주석문법(_In_, _In_opt_)
    - 특수한 주석으로 파라미터가 어떻게 사용될지를 알려준다.
    - 상세내용

        [SAL 주석](https://docs.microsoft.com/ko-kr/cpp/c-runtime-library/sal-annotations?view=msvc-160)

```cpp
//단축키 테이블 정보
HACCEL hAccelTable = LoadAccelerators(hInstance, MAKEINTRESOURCE(IDC_CLIENT));

MSG msg;

// GetMessage의 특징
// 메세지 큐에서 메세지가 없으면 메세지가 확인될때까지 대기한다.(종료되지않는다)
//
while (GetMessage(&msg, nullptr, 0, 0))
{
  if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
   {
	     TranslateMessage(&msg);
	     DispatchMessage(&msg);
   }
}

return (int) msg.wParam;
```

- 메시지방식을 통해 윈도우에서 발생한 이벤트들을 처리한다.
- 운영체제에는 메세지큐라는게 존재하는데 메세지를 차례대로 저장하여 FIFO방식으로 출력한다.
- 프로그램은 그 메시지내에 속한 handle이 자신인것만 가져와 차례대로 처리한다.
- 이벤트는 루프를 통해서 처리하는데 프로그램 종료 이벤트가 들어올떄까지 무한히 반복한다.

### window procissor

---

- 콜백 형태로 호출되는 메세지 처리 함수
- 메시지별로 case를 나누어 처리를 할 수 있도록 만들어져 있다
- 사용자가 처리하지 않을 메시지들은 DefWindowProc라는 함수를 호출해 처리하게 해줄 수 있다.

## 윈도우 좌표

---

- 윈도우의 작업영역내에서 위치를 표현한다.
- 단위는 픽셀을 사용한다.

## Window 객체

---

- CreateWindow를 통해 생성되는 윈도우 객체는 운영체제에서 관리하는 객체이기 때문에 프로그래머가 직접 내용을 확인하거나 수정을 할 수 가 없다.
- 운영체제에서 관리되는 객체는 ID를 프로그래머에게 제공해주어서 ID를 통해 핸들링을 할 수 있도록 한다.
- 윈도우에서 제공하는 기본적은 함수에 handle구조체를 넘겨주어 컨트롤하는 방식이다.

## 윈도우 프로시저 핸들

---

```cpp
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    switch (message)
    {
    case WM_COMMAND:
        {
            int wmId = LOWORD(wParam);
            // 메뉴 선택을 구문 분석합니다:
            switch (wmId)
            {
            case IDM_ABOUT:
                DialogBox(hInst, MAKEINTRESOURCE(IDD_ABOUTBOX), hWnd, About);
                break;
            case IDM_EXIT:
                DestroyWindow(hWnd);
                break;
            default:
                return DefWindowProc(hWnd, message, wParam, lParam);
            }
        }
        break;
    case WM_PAINT:
        {
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hWnd, &ps);//Device Context (그리기)

            Rectangle(hdc, 10, 10, 110, 110);

            EndPaint(hWnd, &ps);
        }
        break;
    case WM_DESTROY:
        PostQuitMessage(0);
        break;
    default:
        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}
```

- WM_COMMAND는 창의 메뉴를 선택할때 호출된다.(파일, 메뉴등..)
- WM_KEYDOWN은 키보드가 눌렸을때 이벤트를 담당한다.
    - VK_xxxx를 통해 화살표나, 시프트, 컨트롤등의 이벤트를 처리한다.
    - 영어자판의경우 아스키코드를 통해 가져올수 있다.
    - 예

        ```cpp
        case WM_KEYDOWN:
            {
                switch (wParam)
                {
                case VK_UP:
        		{
        			int a = 0;
        		}
                    break;
                case 'D':
                {
                    int a = 0;
                }
                    break;
                default:
                    break;
                }
            }
                break;
        ```

- WM_L or R MOUSE_BUTTON을 통해 마우스 입력도 가져올수있다.
    - lParam변수에는 각각 2byte씩 마우스의 클릭좌표로 가져올수 있다.
    - LOWORD(),HIWORD()를 통해 각각 x와 y좌표를 가져올수 있다.
- WM__PAINT이벤트가 호출되면 화면을 그린다.
- WM_PAINT는 무효화영역이 발생되었을때 호출된다.
- 무효화영역이 대표적으로 생기는 경우는 창을 내렸다. 다시 활성화했을때이다
    - 내부적으로 이미지데이터를 가지고 있기때문에 예전처럼 창이 가려진것으로 무효화영역이 발생하지 않는다.
- 각각의 handle 구조체들은 정수타입의 변수로 운영체제로 부터 id를 받게된다.
    - 왜 구조체들이 구분되어있을까?
        - id만을 받지만, 각 id가 어떤것을 다룰지, 운영체제는 단순히 숫자만 알려주기 때문에 사용자 입장에서 그걸 구분할 수 가 없다.
        - 따라서 사용자의 편의와 잘못된 사용을 방지하기위해 구조체로 구분하여 제공한다.
    - Device Context?
        - 화면에 렌더링을 수행하는데 필요한 Data집합(DirectX할때도 나오는 개념)
        - 어디에 그릴지, 어떻게 그릴지에 대한 정보를 가지고 있다.
        - 그렇기 때문에 생성시 현재 window정보와, paintstruct정보를 받아서 생성된다.
    - GetStockObject()
        - GDI에서 자주사용하는 펜이나 브러시를 미리 생성해놓았는데, 이걸 가져오는 함수
        - MSDN에서 목록을 확인 할 수 있다.
        - 대표적으로 HOLLOW Brush(투명 브러쉬)를 사용할때 이용한다.
    - Device Context예제 - 펜설정하기& 브러쉬설정하기

        ```cpp
        			//펜을 만들어 device context를 지급
                    HPEN hPen = CreatePen(PS_SOLID,0,RGB(255,0,0));
                    HBRUSH hBrush = CreateSolidBrush(RGB(0, 0, 255));

                    //void포인터로 반환해 필요한 타입으로 캐스팅
                    HPEN hDefalutPen = (HPEN)SelectObject(hdc, hPen);
                    HBRUSH hDefaultBrush = (HBRUSH)SelectObject(hdc, hBrush);

                    Rectangle(hdc, 10, 10, 110, 110);

                    SelectObject(hdc, hDefalutPen);
                    SelectObject(hdc, hDefaultBrush);
                    //필요없는 오브젝트 삭제
                    DeleteObject(hDefaultBrush);
                    DeleteObject(hPen);
        ```