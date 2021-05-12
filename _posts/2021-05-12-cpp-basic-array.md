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