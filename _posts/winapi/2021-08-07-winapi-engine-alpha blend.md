---
layout: post
title:  "[C++][WinAPI] Create Game Engine -16 AlphaBlend"
summary: "winapi game engine"
author: moon
date: '2021-08-07 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/alphablend
usemathjax: true
---
## Alpha Blending

---

- 흔히 색의 투명도를 말하는데, 이 값에 따라 오브젝트가 반투명해지기도 하고, 진해지기도하고 색이 바뀌기도 한다.
- 공식은 RGB값에 alpha값을 곱하고, 1에서 alpha값을 뺀 나머지 값을 다른 RGB값에 곱하여  더해주면 AlphaBlending이완성된다.

    $(R,G,B)*alpha +(R`,G`,B)*(1-alpha)$

## WinApi 에서 AlphaBlending 사용

---

- 기본적으로 bmp파일에는 alpha값이 존재하기 않기 때문에, 32비트짜리 이미지로 바꿔줘야한다.
- 알파 블렌드 함수를 이용해야하는데 기능이 매우 제한적이라 알파를 조절하는데 정도만 의미가 있다.
    - 아래 코드 처럼 BLENDFUNCTION 구조체를 채워 마지막 인자로 넘겨줘야한다.
    - 각각이 설정값의 의미는 MSDN을 참조해서 작성하면 된다.

    [BLENDFUNCTION (wingdi.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/api/wingdi/ns-wingdi-blendfunction)

    ```cpp
    BLENDFUNCTION bf = {};
    	bf.BlendOp = AC_SRC_OVER;
    	bf.BlendFlags = 0;
    	bf.AlphaFormat = AC_SRC_ALPHA;
    	bf.SourceConstantAlpha = m_animator->GetAlpha();

    	AlphaBlend(
    		_dc
    		, (int)(pos.x - m_frames[m_currentFrame].slice.x / 2.f)
     		, (int)(pos.y - m_frames[m_currentFrame].slice.y / 2.f)
     		, (int)(m_frames[m_currentFrame].slice.x)
     		, (int)(m_frames[m_currentFrame].slice.y)
     		, m_tex->GetDC()
     		, (int)(m_frames[m_currentFrame].lt.x)
     		, (int)(m_frames[m_currentFrame].lt.y)
     		, (int)(m_frames[m_currentFrame].slice.x)
     		, (int)(m_frames[m_currentFrame].slice.y)
    		, bf
    	);
    ```

    ### FADE IN, FADE OUT

    ---

    - 화면전환효과로, 서서히 어두워지고, 서서히 밝아지면서 화면이 바뀌는 효과를 말한다.
    - 알파블레딩을 통해 만들수 있는 간단한 효과중 하나이다.
    - 검은색 화면의 알파값을 조절해서 점점 투명하게 했다가 다시 불투명하게 해주는 효과를 반영해주면된다.
    - 이런 화면의 텍스쳐는 굳이 이미지를 부르기보단 직접 만들 수 있고, 이 기회에 Core에 있는 이중버퍼링 코드를 수정해준다.
    - 텍스쳐를 만들게 되면 기본적으로 RGB값이 0이기때문에 굳이 색상을 설정해줄 필요는 없다.
    - 두 효과다 일정시간동안 진행되는 기능이기때문에 누적시간과 최종 진행목표시간이 필요하다.
    - 시간이 지남에따라 텍스쳐의 alpha를 0~255로 변화시켜주면된다.
    - 여러 효과를 순차적으로 진행하고 싶다면, Queue를 이용해 FIFO형식으로 만들면 된다.

    ```cpp
    void CCamera::Render(HDC _dc)
    {
    	//필요한 부분만 렌더링
    	
    	if (m_camEffects.empty())
    		return;
    	camEffect& ef = m_camEffects.front();
    	ef.currentTime += fDT;

    	float ratio = ef.currentTime / ef.duration;
    	if (ratio < 0.f)
    		ratio = 0.f;
    	if (1.f < ratio)
    		ratio = 1.f;
    	int alpha = 0;

    	if (CAMERA_EFFECT::FADE_OUT == ef.effect)
    	{
    		alpha = (int)(255.f * ratio);

    	}
    	if (CAMERA_EFFECT::FADE_IN == ef.effect)
    	{
    		alpha = (int)(255.f * (1 - ratio));
    	}
    	if (CAMERA_EFFECT::VIBRATION == ef.effect)
    	{
    		return;
    	}
    	BLENDFUNCTION bf = {};
    	bf.BlendOp = AC_SRC_OVER;
    	bf.BlendFlags = 0;
    	bf.AlphaFormat = 0;
    	bf.SourceConstantAlpha = alpha;

    	AlphaBlend(
    		_dc
    		, 0, 0
    		, m_veilTexture->Width()
    		, m_veilTexture->Height()
    		, m_veilTexture->GetDC()
    		, 0, 0
    		, m_veilTexture->Width()
    		, m_veilTexture->Height()
    		, bf
    	);
    	if (ef.duration < ef.currentTime)
    	{
    		m_camEffects.pop_front();
    		return;
    	}
    }
    ```