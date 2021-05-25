---
layout: post
title:  "[C++] 문자와 문자열"
summary: "문자와 문자열"
author: moon
date: '2021-05-24 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, basic, for, while, function
permalink: /blog/cpp/basic/string
usemathjax: true
---

### 문자

---

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

    - wchar_t로 배열을 선언해 문자열을 만들때 L을 붙여줘야함
    - 문자열의 끝을 알리기 위해 escape(‘\0’)를 적어야함
    - 그렇기 때문에 글자의 개수 +1의 공간을 차지
    - 배열 선언과 포인터 선언의차이
        - 배열로 문자열을 저장할경우 문자를 배열에 복사하여 사용하는 형식
        - 포인터로 선언할경우 문자열의 시작인 'a'의 시작주소를 할당
            - 왜 const를 선언해야하는가
                - 문자열은 ROM(Read Only Memory)에 저장되는데 이름 그대로 읽기전용 메모리이다
                - const가 없을경우 이 읽기전용 메모리에 접근하여 값을 변경을 하게되고 이걸 운영체제(플랫폼)가 이를 막으면서 프로그램이 고장나게되버림