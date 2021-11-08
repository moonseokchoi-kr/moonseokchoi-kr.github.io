---
layout: post
title:  "[C++][Direct2D] Create Game Engine -2 Game Engine Structure"
summary: "Direct2D game engine"
author: moon
date: '2021-09-06 12:35:23 +0900'
category: Direct2D
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, Direct2D, ui, grapic,engine
permalink: /blog/cpp/Direct2D/engine/Game Engine Structure
usemathjax: true
---
## 게임엔진의 구조

---

### 게임엔진과 프레임워크의 차이

---

- 이전에 WINAPI로 만들었던 프로그램은 게임엔진이라 부를 수 없다.
- 왜냐하면 코드에서 게임엔진과, 게임콘텐츠를 완전히 분리할수가 없기 때문이다.
- 이러한 방식으로 구성을 하면 매번 게임을 만들때마다 엔진을 새로 만들어야하는 문제가 발생한다.
- 게임엔진은 이러한 구조를 가지지 않는다 이러한 구조는 프레임워크에 가깝다

- 게임엔진은 콘텐츠와 엔진의 기능이 완전히 구분된다.
- 이렇게 구분된 각각의 모듈을 라이브러리라고 부른다.
- 즉 게임엔진은 라이브러리로 구성되어 필요한 부분만 가져와 이용하는 형태로 구성된다는 의미이다.
- 게임엔진에서 중요한 것은 **'모듈화'구성이다**

### 프로젝트 디렉토리 정리

---

- Solution Dir
    - Project
        - Client
            
            
        - Engine
    - OutputFile
        - bin
        - bin_debug
    - External
        - include
            - Engine
        - Library
            - Engine
- 현재 프로젝트 실행을 위해서 라이브러리와 실제 프로그램 프로젝트 디렉토리가 다르다 보니 매번 빌드하고나면 복사를 해야하는 문제가 있다.
- 이러한 문제를 간단하기 위해서

### 게임엔진 코어 설계

---

- 정적할당 방식으로 싱글톤이 아닌 동적할당 방식의 싱글톤을 사용한다.
- 정적할당은 메모리 해제의 순서를 제어할 수 없기 때문에 구현의 자유도가 낮아지지만 실수의 여지가 없어지고, 동적할당은 자유도가 높아지는대신 메모리 해제를 하지않아 문제가 발생할 여지가 있다.
- 즉 메모리해제를 확실히 시킬 수 만 있다면 동적할당이 좀더 능동적인 코어 설계를 가능하게 한다.

### 동적할당의 종속성

---

- 매니저들이 싱글톤으로 설계되어도 서로 종속 되는 경우가 있다. 대표적으로 path와 resource는 path가 먼저 정해져야 resource를 가져올수 있기때문에 path에 종속되어있다고 볼 수 있다.
- 이러한 종속관계에서는 생성과 소멸이 역순이다.(생성이 먼저라면 소멸이 나중)
- 이러한 종속관계를 이용하면 동적할당의 메모리 해제를 신경쓰지 않고 능동적인 프로그래밍이 가능해진다.
- 이를위해 사용할수 있는 함수가 존재한다.

[atexit](https://docs.microsoft.com/ko-kr/cpp/c-runtime-library/reference/atexit?view=msvc-160)

- atexit 함수는 종료시 실행될 함수를 스택으로 관리하여 최근에 집어넣은 함수부터 실행하게된다.
- 아래처럼 싱글톤을 생성하게되었을때 atexit에 소멸하는 함수를 넣어놓으면 실수할 여지를 줄일수 있게된다.
    
    ```cpp
    //싱글톤 풀코드
    template<typename T>
    class CSingleton
    {
    private:
    	static  T* g_pInst;
    public:
    	static T* GetInst()
    	{
    		if (nullptr == g_pInst)
    		{
    			g_pInst = new T;
    		}
    		return g_pInst;
    	}
    private:
    	static void Destroy()
    	{
    		if (nullptr != g_pInst)
    		{
    			delete g_pInst;
    			g_pInst = nullptr;
    		}
    	}
    
    protected:
    	CSingleton () 
    	{
    		void(*pFunc)(void) = &CSingleton::Destroy;
    		atexit(pFunc);
    	};
    	~CSingleton(){};
    
    };
    
    template<typename T>
    T* CSingleton<T>::g_pInst = nullptr;
    ```
    
- 또한 템플릿으로 동적할당 방식의 싱글톤을 만들어 놓으면 매번 위의 코드를 반복하지 않고 상속으로 한번에 해결할 수 있게된다.

### DirectX의 스마트포인터

---

- 스마트포인터를 사용하는 이유
    - 현재 다이렉트 X객체를 생성한뒤 해제를 하기 위해 수동적으로 릴리즈를 하고 있는데 이러한 방식은 프로그램이 복잡해지면 실수로 해제를 못하는경우가 생겨 문제를 야기 할 수 있다.
    - 스마트포인터를 이용하는경우, 알아서 래퍼런스 카운트를 체크해 0이되면 소멸자가 실행되므로 프로그래머가 더이상 신경쓰지 않아도 된다.
- 스마트 포인터를 사용할때 유의할점
    - 스마트포인터의 포인터는 실제 가지고 있는 원소의 포인터가 아니다.
    - 이중포인터를 사용할때 참조를 넘기는 방식을 이용하면 문제가 발생한다.
    - 이러한 점을 대비하기 위해 스마트포인터에는 원본을 가져올 수 있는 함수들이 존재한다.