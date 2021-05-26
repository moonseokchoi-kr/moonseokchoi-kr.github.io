---
layout: post
title:  "[C++] 문자와 문자열"
summary: "문자와 문자열"
author: moon
date: '2021-05-26 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, basic, for, character, string
permalink: /blog/cpp/basic/string
usemathjax: true
---

### 문자

---

- 메모리 영역 관련 포스팅확인
- 문자의 자료형
    - char(character)
        - 1바이트크기의 문자 자료형
        - 문자 1개만 저장가능
        - 문자를 정수형태로 표현한다(ASCii Code)
    - wchar_t(wide character)
        - 2바이트크기의 문자 자료형
        - 문자열을 저장가능

- 문자열

    ```cpp
    wchar_t sWchar[10]= L"abcdef";
    const wchar_t* pWchar = L"abcedf";
    ```

    - 문자열의 끝을 알리기 위해 escape(‘\0’)를 적어야함 그렇기 때문에 글자의 개수 +1의 공간을 차지
    - 문자는 주소로 취급하기 때문에 주소를 요구하는 함수에 바로 넣을수 있음
    - 배열 선언과 포인터 선언의차이
        - 배열로 문자열을 저장할경우 문자를 배열에 복사하여 사용하는 형식
        - 포인터로 선언할경우 문자열의 시작인 'a'의 시작주소를 할당
            - 왜 const를 선언해야하는가
                - 문자열은 ROM(Read Only Memory)에 저장되는데 이름 그대로 읽기전용 메모리이다
                - const가 없을경우 이 읽기전용 메모리에 접근하여 값을 변경을 하게되고 이걸 운영체제(플랫폼)가 이를 막으면서 프로그램이 고장나게되버림
    - 멀티비아트와 유니코드

        ```cpp
        //멀티바이트
        char sChar[10] = "LSOFJ한글"
        //유니코드 L:유니코드 표시
        wchar_t sWchar[10] = L"LSOFKDF한글"
        ```

        - 멀티바이트
            - ASCii를 통해서 문자를 표현했는데, 128개만으로 문자를 표현하는데 한개가 있어 이후 바이트를 추가하여 2바이트, 3바이트를 이용해 문자를 표현하는 방식이다.
        - 멀티바이트의 단점
            - 어떤건 1바이트, 어떤문자는 2바이트등 문자마다 코드 페이지가 다르기 때문에 이를 제대로 확인해서 사용해야 한다. 만약 잘못사용할 경우 글자가 모두 깨져버려 글을 알아볼수 없게된다.
        - 유니코드
            - 모든문자를 표현하기위해 만들어진 2byte크기의 코드 집합체 이다. (문자마나 유니코드가 있다.) 모든 문자를 2byte로 표현할 수 있기 때문에 멀티바이트에서 발생한 문제점을 보완가능하다.
    - 문자열과 관련된 함수

        ```cpp
        #include <wchar.h>
        //문자열길이
        wcslen(L"abcdef)
        //문자열 이어붙이기
        wcscat_s(L"abc",10,L"def")
        //문자열 비교
        wcscmp(L"abc",L"def")
        ```

        - wcslen
            - 문자열을 입력하면 길이를 확인할 수 있다. const wchar_t*를 요구한다.
        - wcscat_s
            - 목적문자열에 소스문자열을 붙히는 방식이다. 중간에 입력하는 사이즈보다 두 문자열의 길이의 합이 더 크면 에러를 리턴한다.
        - wcscmp
            - 문자열 두개를 비교하여 같은면 0, 서순이 왼쪽이 더 크면 1, 오른쪽이 더 크면 -1을 반환한다.