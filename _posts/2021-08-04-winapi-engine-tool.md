---
layout: post
title:  "[C++][WinAPI] Create Game Engine -13 Tool"
summary: "winapi game engine"
author: moon
date: '2021-08-04 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/tool
usemathjax: true
---
## Tile

---

게임내 맵을 구성하는 하나의 작은 단위로 2D게임에서 맵을 제작할때 사용한다.

이런 타일의 제작할때에는 2의 승수(2,4,8,16,32,64.)이런식으로 지정해서 만드는게 추후 GPU를 이용했을때도 연산이 더 유리하다.

Map을 제작하는 툴의 기능

---

1. 마우스를 이용하여 타일을 배치
2. 만든 맵을 save & load

## 실시간으로 타일의 갯수를 확인하기

---

- 다이얼로그
    - main 함수내에 윈도우 프로시저 함수 내에 ABOUT버튼을 누르면 정보창이 나오도록 설정되어있다.
    - 정보창이 나타나면 해당 프로시저가 실행되고 있으므로 메인창에 대한 이벤트입력이 되지 않는다(중지되지는 않음)
    - 다이얼로그 상자도 하나의 윈도우 이기때문에 그 내부에 있는 메세지처리 때문에 Focuse가 다이얼로그 상자에 집중된다.(about함수)
    - 이 정보창을 여는 함수인 DialogBox는 리소스를 요구하는데 이 리소스를 찾아보면 리소스폴더내에 저장되어 있는 이미지 데이터이다.
- 모달방식
    - 포커싱을 기반으로 활성화된 창에 모든포커싱을 가져가는 방식
- 모달리스 방식
    - CreateWindow로 창을 생Di성해 하나의 이벤트 처리 함수를 이용해 여러창의 이벤트를 처리하는 방식 포커싱이 하나에 집중되지 않는다.

### Dialog 생성하기

---

- 다이얼로그는 리소스뷰를 통해 생성하게 되면 자동으로 ID가 부여된다.
- 이렇게 지정된 ID는 resource.h에 저장된다.
- 둘은 한쪽이 열면 한쪽이 참조하는 형태이기 때문에 동시에 열수 없다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2793e665-ce2a-439c-8a58-6c2e11d87a3a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2793e665-ce2a-439c-8a58-6c2e11d87a3a/Untitled.png)

이런식으로 다이얼로그 창을 변경하여 생성가능하다

- 다이얼로그 안에 들어있는 버튼도 결국 윈도우이다. 즉 화면을 구성하는 UI들은 다 기능이 다른 윈도우라고 생각할 수 있다.
- 윈도우를 생성했다면 프로시저를 통해 다이얼로그가 할일을 처리할 수 있는데, 이때 반환되는 값으로 메인함수에서  확인이 눌렸는지, 취소를 눌렸는지 확인 할 수 있다.