---
layout: post
title:  "[C++] Template"
summary: "템플릿기능과 함수, 클래스, 구조체"
author: moon
date: '2021-06-02 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, template, class, structure
permalink: /blog/cpp/basic/template
usemathjax: true
---
## 함수 템플릿

---

- 왜 사용할까?
    - 기능은 똑같은데 받아야하는 인자만 다를경우 이를 편하게 하기위해 사용

    ```cpp
    //둘의 기능은 같지만 자료형이 달라 같은 함수를 여러번 선언해야한다.
    int add(int a, int b)
    {
    	return a+b;
    }

    float add(float a, float b)
    {
    	return a+b;
    }

    //템플릿 사용
    template<typename T>
    T Add(T a, T b)
    {
    	return a+b;
    }
    ```

    - 템플릿함수는 템플릿으로 사용할 자료형이 정해지면 그때 생성된다.(코드영역이아님)
    - 함수의 타입이 정해져도 함수라 명칭하지않고 함수템플릿이라 부른다.

## 클래스 템플릿

---

- 왜사용할까?
    - 비슷한 목적과 구조를 가진 클래스(자료구조를 이용한다던가..)들이 여러개를 생성해야할때 반복적인 노가다를 줄여주기 위해 사용(Generic programming)
- 클래스템플릿은 모두 head파일에 작성되어야한다
    - 선언과 구현이 분리되어서 구성이 되면, 템플릿이 구현되어있는 부분을 연결할 방법이 없기 때문에 선언과 함께 구현을 적어야한다.(템플릿은 사용자가 어떤 타입으로 사용해야 결정이 되기 때문에 cpp를 특정할 수 없음)

    ```cpp
    template<typename T>
    Class Test{}

    Test; //템플릿
    Test<int>; //클래스명
    ```

    - 템플릿으로 선언된 클래스 네임은 템플릿이고, 자료형까지 선언된게 클래스명이된다. 자료형이 다르면 서로 다른 클래스에 해당한다.

## Generic Programming

---

- 코드의 재사용을 목적을 **템플릿(Templet)** 을 사용하여 만드는 프로그래밍 기법
- 객체지향 프로그래밍에 속하지는 않지만 객체지향 프로그래밍과 같이 사용하면 강력한 효과를 발휘한다.
- 템플릿은 매개변수화를 이용하여 값뿐만 아니라 데이터 타입까지도 매개변수화를 시키는것
- 함수에서 알고리즘을 구현할때 입력타입에 구애 받지않고, 구현을 할때 주로 사용한다.

    ```cpp
    //sqrt()의 구현
    #define _GENERIC_MATH1R(FUN, RET, CRTTYPE) \
    extern "C" _Check_return_ CRTTYPE RET __cdecl FUN(_In_ double); \
    //매개변수화 템플릿기능적용
    template<class _Ty, \
    	class = _STD enable_if_t<_STD is_integral_v<_Ty>>> _NODISCARD inline \
    	RET FUN(_Ty _Left) \
    	{ \
    	return (_CSTD FUN(static_cast<double>(_Left))); \
    	}
    ```

- 템플릿 기능은 매우강력해서 좋지만, C++언어의 템플릿 선언이 매우 복잡하기 때문에 의도적으로 회피하는 사람들이 있다.

- 템플릿을 사용하지 않고 만들경우 생길 수 있는 위험성(Templete 프로젝트 참고)
    1. GameBoard는 항상 원소를 포인터로 저장해야함(값은 저장할 수 없음)
    2. 타입안정성이 떨어진다, 저장했던 셀을 요청하기 위해 at을 호출해도 unique_ptr<GamePiece>로만 리턴함 그래서 ChessPiece로 다운캐스트해야만 사용가능
    3. int나 double같은 기본타입은 사용할 수 없다.(GamePiece를 상속한 타입만 가능)
    4. 상속을 구현했기 때문에 엉뚱한 값을 저장할 가능성이 있음(chess 판에 tictactok게임말을 저장한다거나..)