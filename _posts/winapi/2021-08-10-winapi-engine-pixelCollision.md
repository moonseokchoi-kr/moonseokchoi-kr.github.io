---
layout: post
title:  "[C++][WinAPI] Create Game Engine -18 Pixel Collision"
summary: "winapi game engine"
author: moon
date: '2021-08-03 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/PixelCollision
usemathjax: true
---
### 픽셀충돌 원리

---

- 보통은 충돌체를 통해 지형을 구성해서 땅을 만들지만, 지형이 울퉁불퉁하거나 지형이 특이해 충돌체로 만들기 힘들 경우가 있다.
- 대표적으로 메탈스러그의 맵들을 보면 알 수 있다.
- 배경이미지와 땅을 나타내는 색(마젠타 색상)을 입힌 이미지 두가지를 준비한다.
- **각각의 비트맵을 만들고, 비트맵에서 색을 가져와 땅을 나타내는 색상이라면 그 위치를 땅이라고 인식하고 충돌처리를 하게된다.**

 

### 픽셀충돌 방법

---

- 원래 winapi 비트맵에 픽셀을 가져오는 함수가 있지만 이 함수를 사용할경우 매우 느려지기 때문에 직접 만드는게 좋다.
- 만들떄 주의할점은 다음과 같다
    - 비트맵은 좌하단부터 0으로 시작한다. 만약 0번째 픽셀에 접근하고 싶다면 비트맵 개수의 -1로가야한다.
    - 비트맵 픽셀 하나의 크기는 4 바이트이다.
- 이제 실제 만들어 보겠다.
- 먼저 Color 구조체를 만든다.

    ```cpp
    struct tColor
    {
    	BYTE red,
    	BYTE green,
    	BYTE blue,
    }
    ```

- 그 다음 비트맵의 정보를 가져온다

    ```cpp
    tColor CTexture:GetPixel(UINT _x, UINT _y)
    {
    //비트맵 정보 가져오기
    	UINT iWidth = m_bitInfo.bmWidth;
    	UINT iHeight = m_bitInfo.bmHeight;
    	_y = iHeight-(y+1);
    //픽셀 개수 계산
    	UINT iWideByte = iWidth * sizeof(tColor);
    //4바이트로 맞춰주기
    	iWideByte += 4 - iWideByte % 4;
    //픽셀 입력
    	BYTE* pByte = (BYTE*)m_bitInfo.bmBits;
    	tColor* pColor = (tColor*)(pByte+iWideByte*(_iY)+(_iX*3));
    	return *pColor
    }
    ```