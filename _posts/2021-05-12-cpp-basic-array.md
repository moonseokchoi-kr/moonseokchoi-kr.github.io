---
layout: post
title:  "[C++] 배열"
summary: "배열"
author: moon
date: '2021-05-06 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, basic, for, while, function
permalink: /blog/cpp/function/
usemathjax: true
---

## 배열
=========================

- 많은 수의 변수를 하나의 변수로 통제하기 위해 사용하는 방법

    ```cpp
    //int 변수 10개를 일일이 선언하여 관리하면 힘들다
    int i =0;
    int ii= 0;
    ---
    int iii...iii =0;

    //배열
    int iArray[10] = {};
    ```

- 배열의 값에 접근 방법
    - 인덱스를 선택하여 접근한다.
    - 배열의 크기가 10이라면 인덱스는 0~9까지 할당된다.

        ```cpp
        int iArray[10] = {};
        //1번째 인덱스
        iArray[0] = 10;

        //인덱스 범위밖 (크키가 할당되어있지 않은 Un-Safe하기 때문에 이용할수 없음)
        //메모리가 할당되어있으면 오류도 안잡힘(이래서 STL을 쓰는거다)
        //디버그시 에러를 잡아주나, Realse시에는 안잡힘으로 매우 신경써줘야함
        iArray[10] = 10;
        ```

    - 배열은 연속적으로 공간이 할당되기 때문에 인덱스 밖에는 어떤값이 할당되어있는지 알수 없음

## 구조체
==============================

- 의미
    - 사용자가 정의한 자료형

        ```cpp
        //구조체의 선언(C 언어 선언방법)
        //typedef struct (구조체이름)
        typedef struct _tagMyST
        {
        	int a;
        	float f;
        } MYST;//=>리넴임(이것으로 호출한다);

        //구조체 선언 (C++)

        struct _tageMyST
        {
        	int a;
        	float f;	
        };
        ```

    - 구조체에서 typedef 키워드는 타입재정의를 의미한다.(기본 타입들에 대한 재정의도 가능)
    - C언어에서는 struct를 명시해야만 구조체로 인식하기 때문에 typedef를 사용해서 이름을 재정의한다.
    - C++에도 C언어 방법이 남아있는 이유는 C언어와의 호환때문에 남아있다.
- 구조체 이용방법

    ```cpp
    struct _tageMyST
    {
    	int a;
    	float f;	
    };

    int main()
    {
    	//배열처럼 선언된 순서에 따라 괄호를 이용해 초기화가능
    	 _tagBMyST f = { 1,2 };
    	//.(닷)을 이용하여 구조체 내부의 데이터에 접근가능
    	f.a = 10;
    	f.f = 20;
    }

    // 원래 함수를 추가하거나 생성자를 추가하는등의 과정도 가능하지만 그건 나중에 할 예정
    ```