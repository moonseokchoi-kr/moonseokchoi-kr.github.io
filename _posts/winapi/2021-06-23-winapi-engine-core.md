---
layout: post
title:  "[C++][WinAPI] Create Game Engine -1 Core Class"
summary: "winapi game engine"
author: moon
date: '2021-06-23 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/core
usemathjax: true
---
## 엔진코어 제작

---

- 게임에서 중요한 역할을 하는 코어 클래스를 제작
- 객체가 여러개 생성되면 안된다. 코어클래스는 게임에서의 중추이기 때문에
- 따라서 Singleton pattern을 기반으로 제작한다.
    - Singleton Pattern
        - 객체가 1개만 생성가능함을 보증하고 싶을때 사용하는 패턴이다.
        - 생성자를 숨겨 생성자를 사용자가 호출 할 수 없도록 한다.
        - 대신 객체를 생성하는 함수를 만들어 제공한다. 아래와 같이 작성해서 사용한다.

            동적할당을 이용한 방법

            ```cpp

            class Singleton
            {
            	public:
            		static Singleton GetInstance()
            		{
            				static Singleton* singleton= nullptr;
            				if(nullptr == singleton)
            				{
            						sigleton = new Singleton();
            				}
            				return sigleton;
            		}
            	private:
            		Singleton();
            		~Singleton();
            }

            //사용

            Singleton single = Sigleton.GetInstance();
            ```

            데이터영역을 이용한 방법

            ```cpp
            // 데이터영역을 이용한 싱글톤 구조
            // 해제를 신경쓰지 않아도 알아서 사라진다.
            // 대부분 매니저 형태를 싱글톤으로 만들기 때문에 이 방법을 많이 사용할 수 있음
            class Singleton
            {
            public:
            	static Singleton* GetInst()
            	{
            		static Singleton core;
            		
            		return &core;
            	}

            private:
            	Singleton() {}
            	~Singleton() {}

            };
            ```

            - 왜 함수 정적변수로 동적할당을 안하나요?
                - 함수로 정적변수로 제작을 하게되면 메모리를 해제할 때 문제가 된다.
                - 함수를 통해 메모리를 해제하게 되는데 정적변수의 메모리는 해제가 안되기 때문에 다시 필요할때 생성이되는게 아닌 이미해제되버린 메모리를 반환해준다.
- 미리 컴파일된 헤더
    - 윈도우에서 제공되는 기능으로 컴파일이 완료된 헤더들을 모아놓은 헤더를 말한다.
    - 컴파일시 미리컴파일된 헤더는 체크를 안하게 된다.
    - 미리컴파일된 헤더를 사용하면 cpp에서 자동으로 참초하게된다.

## 생성된 코어 클래스

---

```cpp
class CCore
{
	SINGLE(CCore);
public:
	//코어클래스를 초기화
	int InitCore(HWND _hwnd, POINT _resolution);
	//초기화후 처리할 과정을 진행
	void Progress();

private:
	CCore();
	~CCore();
private:
	HWND m_hWnd;
	POINT m_ptResolution;
};
```

## 초기화처리 방법

---

```cpp
if (FAILED(CCore::GetInst()->InitCore(g_hwnd, POINT{ 1280, 768 })))
	{
		MessageBox(nullptr, L"Core 객체 초기화 실패", L"ERROR", MB_OK);
		return FALSE;
	}
```

- 기존의 boolean 타입이 아닌 HRESULT 타입을 이용하여 진행(windows.h를 포함해야사용가능)
    - 이제 앞으로 이용될 라이브러리들이 대부분 HRESULT를 사용하기 때문에이용
- 만약 에러발생시 메세지 박스가 나오도록 설정