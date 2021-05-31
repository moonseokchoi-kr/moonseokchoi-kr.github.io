---
layout: post
title:  "[C++] Radom Library"
summary: "랜덤"
author: moon
date: '2021-05-31 12:35:23 +0900'
category: cpp
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, library, Random, function
permalink: /blog/cpp/basic/library/random
usemathjax: true
---
## rand()

---

- c기본라이브러리인 <time.h>애서제공
- 랜덤숫자 페이지에서 순차적으로 숫자를 읽어오는 방식이라 완벽한 랜덤이 아님

```cpp
#include <time.h>

rand();
```

rand()

---

- 시드값을 가지는 랜덤함수로 시드값이 달라지면 랜덤값이 달라짐
- 보통 시간을 시드값으로 사용한다

```cpp
#include<time.h>

srand(time(nullptr));
```