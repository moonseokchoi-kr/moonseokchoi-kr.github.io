---
layout: post
title:  "[C++] 동적할당"
summary: "동적할당"
author: moon
date: '2021-05-27 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, basic, pointer, allocate, malloc
permalink: /blog/cpp/basic/allocate
usemathjax: true
---

## 동적할당

---

- 의미
    - 프로그램을 실행하면서 메모리를 할당하는 방법
    - 힙영역의 메모리가 할당된다.
- 장점
    - 런타임중에 대응가능
- 단점
    - 사용자가 직접 메모리 관리해야함
    - memory-safety가 보장되지 않음(나중에 배우는 스마트포인터를 쓰면 어느정도 해결가능)
- 방법
    - malloc이용
        - 동적할당 함수로 사용자가 할당받고싶은 공간의 크기(byte)를 입력
        - void*가 리턴값이기 때문에 사용자가 캐스트로 용도를 정해야한다.

            ```cpp
            //기본형식
            malloc(100)

            //실사용
            int* pInt = (int*)malloc(100);

            //메모리해제
            free(pInt)
            ```

        - 왜 메모리를 해제해야할까?
            - 동적할당의 경우 프로그램이 힙어디에 어느정도의 메모리를 할당하게될지 알 수가 없어 자동으로 메모리를 해제해주지 않는다. 그렇기 때문에 프로그래머가 신경을 써야한다.
        - 왜 malloc은 void*일까?
            1. 사용자에게 자유를 보장하기 위해서인게 크다, 

            사용자는 동적할당을 어떠한 목적으로 사용할지에 따라 필요한 데이터타입이 다른데, 만약 malloc을 특정 데이터타입으로 제한해 버리면 사용자의 목적에 따라 바꿔쓸 수 없으므로 void*로 선언되어있다.
            2. malloc은 오직 주소값만 반환한다.

            malloc은 동적할당된 메모리공간의 주소값만 반환하면 된다. 그렇기 때문에 주소값을 통해 무언가를 접근하는걸 malloc이 할필요가 없기때문에 모든 주소값은 저장할 수 있는 void*로 반환한다.

## 가변배열(ArrayList)

---

- 의미
    - 런타임중에 크기를 결정하는 배열
    - 힙영역을 사용해 메모리를 할당함
- 왜 일반적으로 가변배열을 못만들까?
    - 지역변수로 선언한 배열이 가변적일 수 없는이유
        - 스택이나 데이터영역은 컴파일러가 자동으로 메모리를 해제하는데 이게 가능한 이유는
        모든 메모리의 크기를 파악하고 있기 때문이다. 그렇기 때문에 배열을 가변적으로 선언하게되면 배열의 크기를 파악할 수 없기때문에제한이 된다.
    - 구조체에서 배열이 가변선언 될수 없는 이유
        - 구조체도 자료형이기때문에 메모리의 크기가 고정적으로 되어야 컴파일러에서 이를 파악하고 관리할 수 있다.
- 가변배열의 선언

    ```cpp
    //Arr.h
    typedef struct _tagArr
    {
    		int* pInt;//주소값저장
    		int iCount;//현재갯수
    		int iMaxCount//최대갯수
    		_tagArr()
    		{
    			pInt = (int*)malloc(sizof(int)*2);
    			iCount = 0;
    			iMaxCount=2;
    		}
    		~_tagArr()
    		{
    				free(pInt);
    				iCount =0;
    				iMaxCount=0;
    		}
    		//재할당
    		private void ReAllocate()
    		{
    				int* pTmp = (int*)malloc(2*imaxCount*sizeof(int));
    				for(int i=0; i<iCount; ++i)
    				{
    					pTmp[i] = pInt[i];
    				}
    				free(pInt)
    				pInt = pNew;
    				iMaxCount *=2;
    		}
    		//[]오버라이딩
    		//추가
    		void add(int _data)
    		{
    				if(iCount == iMaxCount)
    					ReAllocate();
    				pInt[iCount++]=_data;
    		}
    }tArr;
    ```

- 동적할당시 주의할점
    - 무조건 필요한 만큼만 메모리를 할당할것
    - 만약 메모리를 잘못 선언할경우 생길수 있는 일
        - 힙손상문제 (Heap Corruption)
        - 다른 객체가 사용하는 메모리 침범으로 인한 crash
        - 가장 문제점은 c++은 사용자의 자유를 위해 메모리 세이프티를 보장하지 않기때문에 이러한 문제에 대한 catch를 해주지 않아 발견이 힘듬

## Linked List(연결형 리스트)

---

[Data Structure & Algorithm](https://www.notion.so/Data-Structure-Algorithm-a798bffa34bb494b9376c02e74a2ad8b)

- 자료구조는 위 링크에 들어가서 확인할것

```cpp
//Node
typedef struct _tagNode
{
	int value;
	struct _tagNode* pNextNode;
}tNode;

//List
typedef struct _tagList
{
	tNode* pHeadNode;
	int iCount;
}tLinkedList;

//리스트초기화
void InitList(tLiknedList* _pList);

//리스트 데이터삽입
void PushBack(tLinkedList* _pList, int _data);
{
	//노드를 위한 동적할당
	
	//리스트가 비었는지 체크
		//비어있으면 첫노드로 링크

	//아니면 리스트의 마지막 노드를 찾아 연결	
}

void RealseList(tLinkedList* _pList);
{
	//지우기전에 다음노드를 저장해놓고 지우기
}

//과제 
void PushFront(tLinkedList* _pList, int _pData){}
```

## 객체

---

- 의미
    - 데이터타입을 가지고 만들어진 주체
    - int a에서 a가 객체에 해당
    - 붕어빵이 객체 붕어빵을 만드는 틀이 자료형이라생각하면 편함

---
