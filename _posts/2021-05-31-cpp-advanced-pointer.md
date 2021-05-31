---
layout: post
title:  "[C++] 포인터심화"
summary: "포인터심화"
author: moon
date: '2021-05-31 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, advanced, pointer, function_pointer, 
permalink: /blog/cpp/advanced/pointer
usemathjax: true
---

## 함수 포인터

---

- 변수처럼 함수의 포인터를 가져올 수 있고 이를 함수포인터라 함
- 함수포인터를 이용하면 함수를 인자로 넘겨 원하는 기능을 선택하여 작동하도록 만들 수 있음
- **인자로 넘겨주기 위해서는 함수에 선언되는 반환타입 파라미터 타입이 똑같아야만 가능**
- 함수는 주소를 얻기위해 참조파라미터를 붙여도 되고 안붙여도 가능하다
- 다른사람이 만든 Callback해줄 함수를 받아서 구현하는형태로 많이 쓰임
- 예

```cpp
void BubbleSort(int* _pData, int)
//위의 버블소트를 인자로 넘길 수 있따.
void Sort(tArr* _pArr, void(*SortFunc)(int*,int)){}

Sort(&s, BubbleSort);
```