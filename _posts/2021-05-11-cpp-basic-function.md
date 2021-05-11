---
layout: post
title:  "[C++] 반복문과 함수 정리"
summary: "반복문과 함수"
author: moon
date: '2021-05-06 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, basic, for, while, function
permalink: /blog/cpp/function/
usemathjax: true
---
## 반복문

- 조건을 만족할때까지 코드를 반복시키는 구문

    ```cpp
    for(/*기본값*/; /*반복조건식*/; /*업그레이드*/)
    {
    }
    for(int i=0; i<10; ++i){}
    //while-do
    while(종료 조건식)
    {
    	
    	//업그레이드
    }
    while(i<10){}

    //do-while
    do
    {
    	 //수행할 로직	
    }
    while(//반복조건)

    do{}while(i<10)
    ```

- 반복조건, 종료조건에는 true나 false를 판단할 수 있는 식이 들어가야함
- 반복문에 사용할 수 있는 예약아(Keyword)
    - continue : 예약어 실행 시점을 기준으로 이후 과정을 무시하고 반복문을 다음순서로 넘긴다.
    - break :  예약어 실행 시점을 기준으로 반복문을 탈출한다.

## 함수

- 프로그램은 함수의 호출로 구성되어있다.
    - main → function → function
        - 함수는 스택으로 관리되어 LIFO(Last In First Out)형태이다. 따라서 마지막 실행된게 먼저 실행되고, 처음들어온게 마지막 실행되어, 호출은 메인에서 시작되지만 종료는 메인이 마지막이 된다.

        ```cpp
        //함수의 선언
        int Add(int left, int right)
        {
        	return left+right;
        }

        int main ()
        {
        	//함수의 호출
        	int iDate = Add(100,200);
        	
        	return 0;
        }
        ```

    - main function
        - main function은 프로그램의 진입점을 담당하는 함수
        - 컴퓨터 메모리 공간을 선점한다.(static개념으로 main은 프로그램 실행을 위해 사전의 구성이되어 메모리에 올라간다)
        - 그렇기 때문에 main함수에 모든 걸 작성하면 엄청나게 메모리를 잡아먹게됨으로 이러한 프로그램은 피해야한다.
    - console function

        ```cpp
        //콘솔
        	// printf
        	printf("Output natural: %d\n", 3);
        	printf("Output number: %f\n", 3.141592);
        	for (int i = 0; i < 10; ++i) 
        	{
        		printf("OutputTest: %d\n", i);
        	}
        	

        	// scanf
        	int iInput = 0;
        	scanf_s("%d", &iInput);
        ```

        - printf
            - 기본적인 콘솔 출력을 위해 사용된다.
            - %d(정수) %f(실수) 이러한 처리기를 사용해서 문장에 포함하여 실수나 정수를 출력할 수 있다.
        - scanf_s
            - 콘솔을통해 사용자의 입력을 받는 함수
            - 입력이 들어올 때까지 무한히 대기한다.
            - 입력을 받아 저장할 값의 참조를 넘겨줘야한다.