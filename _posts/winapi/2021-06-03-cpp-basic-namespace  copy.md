---
layout: post
title:  "[C++] namespace & STL"
summary: "랜덤"
author: moon
date: '2021-06-03 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, stl, namespace
permalink: /blog/cpp/basic/library/stl
usemathjax: true
---
## 네임스페이스

---

- 모든 식별자(변수, 함수, 형식 등의 이름)가 고유하도록 보장하는 코드 영역을 정의한다
- 동일한 변수명의 중복을 방지하기위해 사용
- 네임스페이스만 다르면 변수명이 같아도 상관없다.
- 사용예

    ```cpp
    #inlcude <iostream>
    std::cout<< "Hello World" << std::endl;

    //만약 접근지정자를 같이쓰기 귀찮다면 아래처럼 써도 된다. 다만 네임스페이스가 제거되기
    //때문에 충돌문제 방지를 못한다.
    using namespace std;

    cout<<"Hello World"<<endl;

    //특정 식별자만 해제(이 방법을 추천)
    using std::cout;
    using std::endl;
    using std::cin;

    ```

## std namespace

---

- C++에서 제공하는 기본적인 함수들을 선언되어있는 이름공간을 말한다.
- iostream, string등 다양한 기본라이브러리들이 정의되어있다.
- cout(console output)
    - std namespace에 선언된 ostream(output stream)의 객체이다.
    - <<연산자를 오버로딩하여 값을 입력받으면 출력한뒤 자기 자신을 리턴한다
    - 그렇기대문에 <<을 연달아 사용하여 출력하는 행위가 가능하다
- cin(console input)
    - std namespace에 선언된 istream(input stream)의 객체이다.
    - >>연산자를 오버로딩하여 변수를 입력받으면 해당 변수에 입력받은 값을 저장한다.
- endl(end line)
    - 위의 2개와 다르게 함수이다.
    - void타입의 함수로 호출시 개행문자를 실행한다.
    - >>연산자를 오버로딩하여 함수포인터를 받는 식으로 구현이 되어있다.


## Standard Template Library

---

- c++에서 제공해주는 표준 템플릿 라이브러리로 다양한 클래스가 제공된다.

- Vector
    - 가변배열과 같으며 템플릿형태로 자료형을 지정하여 사용한다.

    ```cpp
    #include <vector>

    vector<int> vecInt;
    //뒤에 집어넣기
    vecInt.push_back(10)
    vecInt.push_back(20)
    vecInt.push_back(30)
    //인덱스에서 가져오기
    vecInt[0] = 100;
    //인덱스에서 가져오기
    vecInt.at(0);
    vecInt.data();
    //현재크기
    vecInt.size();
    //최대크기
    vecInt.capacity();
    ```

- List
    - 연결리스트의 기능을 제공하는 템플릿 클래스

    ```cpp
    #include<list>

    list<int> listInt;

    listInt.push_back(5)
    listInt.push_front(1)
    ```

- iterator
    - 연결리스트 클래스 내부에 선언된 이너클래스
    - 연결리스트의 헤드를 받아서 설정
    - 반복자기능을 수행하여 리스트를 내부를 순회할때 이용

    ```cpp
    list<int>::iterator iter = listInt.begin();
    //값을참조(역참조가 아닌 연산자오버로딩임)
    int iData= *iter;
    ```