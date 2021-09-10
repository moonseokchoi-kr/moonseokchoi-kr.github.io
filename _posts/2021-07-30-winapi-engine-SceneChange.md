---
layout: post
title:  "[C++][WinAPI] Create Game Engine -10 Scene Change"
summary: "winapi game engine"
author: moon
date: '2021-07-30 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/scenechange
usemathjax: true
---
- Scene Manager에서 장면전환을 담당한다.
- 이를 위해 새로운 함수를 만든다

    ```cpp
    void ChangeScene(SCENE_TYPE _eScene)
    {
    	//현재씬을 다음씬으로 교환
    }
    ```

- 이제 이 함수를 업데이트에서 호출 하면 끝일까?
    - 아니다, 바로 전환할경우 문제가 있다.
        1. 어느시점에 바뀌느냐에따라 갑자기 호출을 부르는게 달라진다.
        2. 기존의 씬을 생성하기위한 함수 호출순서가 뒤죽박죽이 되어 오류가 발생한다.
    - 즉 이 문제를 해결하기 위해서는 모든 함수의 호출 이 끝난 시점 이후에 바꿔야함으로 지연처리를 해야한다(이벤트 처리)
- Scene Change Event정의
    - 이벤트에서 알려줘야 할것
        - 이벤트 타입과 바꿀 씬의 타입
        - 왜 다음 씬을 알려주지 않아도 되는가?
            - 어차피 씬매니저에서 씬의 관계를 이미 다 알고 있기 때문에 씬의 타입만 알려주면 된다.
    - 이벤트 정의가 끝났으면 이벤트매니저를 통해 일괄 처리를 하게 된다.
    - 이벤트 매니저를 통해 씬매니저의 씬변경함수가 불리게된다.

- ChangeScene의 진행과정
    1. 현재 씬의 탈출과정 진행 
        - 씬에 저장된 모든 오브젝트를 삭제

            ```cpp
            //앞으로 자주 이용할것이기 때문에 템플릿으로 선언
            //헤더에 구현한다.
            template<typename T>
            void Safe_Delete_vec(vector<T>& _vec)
            {
            	for(size_t i=0, i<_vec.size(); ++i)
            	{
            		delete _vec[i];
            	}
            	_vec.clear();
            }
            ```

            - 이 함수를 모든 오브젝트에 대해 반복 호출하여 모두 삭제한다.
    2. 현재 씬을 다음씬으로 설정
    3. 바뀐 씬의 초기화 과정 진행

- 생각해보기
    - 씬이 변경될때 이전 씬에 존재하는 오브젝트를 가져오는 것
        - intent함수를 구현하여 객체를 전달 이전 씬에 존재 했던 객체를 intent함수를 만들어 씬이 변경되어 삭제되기 전에 다음씬에 보내준는 방식으로 구현하면 해결할 수 있다.