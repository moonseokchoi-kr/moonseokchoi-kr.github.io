---
layout: post
title:  "[C++] ponter Advanced"
summary: "memory and pointer"
author: moon
date: '2021-06-15 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, memory, pointer, smart_pointer
permalink: /blog/data-structure/graph
usemathjax: true
---
---

## 메모리 작동과정

---

```cpp
int i = 7
```

- 위와 같은 변수를 자동변수(automatic variable)이라고 부르며 스택에 저장된다. 프로그램의 실행흐름에 따라 메모리가 자동 할당, 해제된다.

```cpp
int* ptr = nullptr
ptr = new int;
```

- new를 이용해 선언을 하게되면 힙메모리가 할당된다.(동적할당)
- ptr은 변수이기 때문에 스택에 존재하고, 가리키는 값은 힙에 존재한다.

```cpp
int** handel = nullptr;
handle = new int;
*handle = new int;
```

- handle 변수는 스택에 존재한다. 그리고 handle의 포인터변수는 힙에 존재하고 그 포인터의 포인터(이중포인터)변수도 힙에 존재하게된다.

## 메모리할당과 해제

---

- 할당과 해제 방법에는 2가지가 있다
    - new & delete
        - C++에서 사용하는 동적할당, new는 메모리를 할당하고, delete는 해제해준다.
    - malloc() & free() ⇒ 절대 사용하지 말것!
        - C에서 사용하던 동적할당으로 C++에서는 사용할 이유가 없다. 특히 객체지향프로그래밍에서 malloc()과 free()는 생성자와 소멸자를 호출하지 않기때문에 이용시 치명적인 오류를 만들어 낼수있다. 그러므로 절대사용하지 말자.
- 메모리 할당에 실패할 경우
    - 메모리가 부족하여 할당에 실패하는 경우가 있다. 이럴경우 실행하던 프로그램이 종료된다.
    - new(nothrow)를 사용하면 메모리가 부족해서 할당이 이루어지지 않아도 프로그램이 종료되지 않는다.

## 포인터의 사용

---

```cpp
//메모리 주소가 7인코드를 만드는코드
char* scaryPointer = (char*)7
```

- 포인터는 단지 메모리 주소이기 때문에 위 코드처럼 남용할 가능성이 높다.
- 포인터는 단지 메모리 주소이기 때문에 강제 캐스팅을 하면 얼마든지 그 용도를 변경할 수 있다.
- 문제는 이러한식으로 사용해도 프로그램 진행에 문제가 없다면 별다른 에러가 출력되지 않고 실행되기 때문에 매우 사용에 주의를 해야한다.

## Smart Pointer

---

- 기존의 C 스타일의 포인터로인한 문제를 방지하기 등장함
- 일반적인 포인터와 달리 지정한 객체가 스코프를 벗어나면 메모리를 자동해제한다.
- 스마트포인터의 종류
    - std::unique_ptr
    - std::shared_ptr
- 두 포인터모두 <memory>헤더에 구현되어있다.

## unique_ptr

---

- 가리키는 대상이 스코프를 벗어나거나 삭제될때 할당된 메모리나 리소스도 자동으로 삭제된다는 점을 제외하면 일반 포인터와 같다.
- unique_prtr로 가리키고 있는 객체는 일반 포인터로 가리킬수 없게된다.
- 특징
    - return문이 실행되거나 익셉션이 발생하더라도 항상 할당된 메모리나 리소스를 해제할 수 있다는 장점이있다.
    - return문을 여러개 작성하더라도 각각의 리소스를 해제할 필요가 없기때문에 코드가 간결해진다.
- 사용방법
    - C++ 14이후
        - std::make_unique<>를 통해 선언

            ```cpp

            //기존 포인터 방법
            Employee* anEmployee = new Employee;
            delete anEmployee

            //smart pointer 방법
            auto anEmployee = make_unique<Emlpoyee>();
            //delete가 나중에 자동으로 호출되기 때문에 작성할 필요없음

            //사용은 일반 포인터와 같다
            if(anEmployee)
            {
            	cout<< "Salary: " << an Employee->salary << endl;
            }

            //배열을 저장하는데도 활용가능
            auto employees = make_uniquer<Employee[]>(10);

            cout<< "Salary: " << employees[0].salary<<endl;
            ```

    - C++ 14 이전
        - std::unique_ptr을 통해선언

            ```cpp
            unique_ptr<Employee> anEmployee(new Employee)
            ```

## shared_ptr

---

- 데이터를 공유할수 있는 포인터
- shared_ptr에 대한 대입연산이 발생할 때 마다 래퍼런스 카운트가 증가한다. 이를 통해 shared_ptr이 가리키는 데이터를 래퍼런스 카운트 만큼 사용하고 있다는 걸 표시한다
- shared_ptr이 스코프를 벗어나면 카운트가 감소한다.
- 래퍼런스카운트가 0이되면 아무도 이 객체를 소유하지 않고 있기때문에 메모리를 해제한다.
- 사용방법
    - unique_ptr가 비슷하게 make_shared를 통해 생성

        ```cpp
        auto anEmployee = make_shared<Employee>();
        if(anEmployee)
        {
        	cout<< "Salary: " << an Employee->salary << endl;
        }

        auto employees = make_shared<Employee[]>(10);

        cout<< "Salary: " << employees[0].salary<<endl;
        ```