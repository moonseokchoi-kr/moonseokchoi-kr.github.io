---
layout: post
title:  "[C++] 조건문과 연산자"
summary: "조건문과 연산자에 대하여"
author: moon
date: '2021-05-06 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, basic, operator, statement
permalink: /blog/cpp/operator/
usemathjax: true
---

# C++ 기본[조건부, 연산자 2]

- switch- case 문
    - if- else if와 매우 유사하다.

        ```cpp
        switch (c)
        {
        	//c의 값이 해당하는 케이스에 따라 실행
        	case 10:
        		break;
        	case 20:
        		break;
        	case 30:
        		break;
        	case 40:
        		break;
        	//어떤 케이스도 해당이 안될때
        	default:
        		break;
        }
        ```

    - 왜 switch - case를 사용하는가?
        - if - else if 로 대부분 사용하지만, switch - case가 더 간결하게 표현이 가능하기 때문에 사용한다. 특히 유저입력에 따라 메뉴가 선택될경우 고정된 값이 들어오는거기 때문에 if-else보다 훨씬 깔끔하게 사용가능하다.
            - 결론: 상황에 따라 맞게 사용하면 된다.
    - break를 사용하는 이유
        - 만약 break가 없을경우 case 10을 실행한뒤 break가 나올때까지 모두 실행한다. 이러한 상황을 방지하기위해 break를 적어야한다.(IDE나 compiler가 잡아주는 오류가 아니기때문에 작성자가 신경써야한다.)
        - break를 적는걸 강제로 제한하지 않는 이유는 or 문장을 처리해야할 수도 있기 때문이다.
        - case 10, 20, 30에 대해서 한번에 처리할 로직이 필요할 수 도 있기때문에 강제하지를 않는다.

- 삼항 연산자
    - (조건식) ? (참일때 수행) : (거짓일때 수행)

        ```cpp
        iTest == 20 ? iTest = 100 : iTest = 200;

        if(iTest == 20)
        {
        	iTest = 100;
        }
        else
        {
        	iTest = 200;
        }
        ```

    - 위에처럼 if-else로 표현할 수 있기 때문에 코딩스타일에 따라서 사용해도 되고 사용안해도 된다.

- 비트 연산자
    - 쉬프트
        - << , >>
        - 각각 왼쪽, 오른쪽으로 비트를 옮긴다. 변수를 n진수의 수라 할떄 왼쪽은  n배만큼 증가, 오른쪽은 1/n배만큼 감소한다.
        - 만약 연산결과로 소수부분이 생기면 버려진다.

            ```cpp
            unsigned char byte = 13;
            	//2^n
            	byte <<= 1;
            	//1/2^n
            	byte >>= 1;
            ```

    - AND(&), OR(&), XOR(&), Reverse(~)
        - 비트연산은 비트 자리에 맞추어 비트연산을 진행한다.
        - AND는 논리연산 AND와 같다 (둘다 True여야 1, 하나만 False라도 False)
        - OR은 논리연산 OR과 같다(한쪽만 1면 결과가 1임)
        - XOR은 같으면 0, 다르면 1이다
        - Reverse는 보수연산으로 각자리의 0과 1을 반전시킨다 ex) 01100 → 10011
    - 비트연산자의 활용(상태확인)

        ```cpp
        //전처리기(매크로)
        // 1. 가독성
        // 2. 유지보수

        //상태저리는 이런식으로 16진수로 표현(찾기 쉽고 구분하기 쉽다.)
        #define HUNGRY 0x1
        #define THIRSTY 0x2
        #define TIRED 0x4
        #define FIRE 0x8

        #define COLD 0x10
        #define BLOODY 0x20
        #define DARKNESS 0x40
        #define POISON1 0x80

        #define POISON2 0x100
        #define POISON3 0x200
        #define POISON4 0x400
        #define POISON5 0x800

        //비트연산자의 활용(상태 표시)

        	unsigned int iStatus = 0;
        	// 0x1
        	iStatus |= HUNGRY;
        	// 0x10
        	iStatus |= THIRSTY;
        	//iStatus = 0x11

        	// 지정한 자리의 비트만 뽑아서 확인가능
        	if (iStatus & THIRSTY)
        	{

        	}

        	//특정 자리 비트 제거
        	//Good Idea!
        	iStatus &= ~THIRSTY;
        ```