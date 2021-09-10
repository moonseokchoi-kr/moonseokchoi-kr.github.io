---
layout: post
title:  "[C++][WinAPI] Create Game Engine -8 late processing"
summary: "winapi game engine"
author: moon
date: '2021-07-28 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/lateprocessing
usemathjax: true
---
- 죽은 몬스터의 삭제, 맵의 전환등 모두에게 한번에 적용해야하는 게임 내 이벤트들이 존재한다.
- 이러한 이벤트들을 충돌처리 같이 즉시 적용하면 어떤 오브젝트는 이러한 이벤트들을 전달받지 못하는 문제가 발생한다.
- 이러한 문제를 해결하기 위해, 프레임단위로 동기화를 시키는 **지연처리**를 담당하는 매니저가 필요하다.
- 매니저를 통해 다음프레임에 적용될 이벤트를 한번에 모아 현재프레임이 끝나면 이벤트를 적용하고 프레임을 재생한다.

## Event Manager 제작

---

- 이벤트 타입 정의
    - 물체의 생성이나, 삭제 , 장면전환등의 이벤트를 구분한다.

        ```cpp
        enum class EVENT_TYPE
        {
        	CREATE_OBJECT,
        	DELETE_OBJECT,
        	SCENE_CHANGE,
        	END,
        };
        ```

- 이벤트 구조체 생성
    - 이벤트를 저장하고 관리할 구조체를 생성한다.

        ```cpp
        struct tEvent
        {
        	EVENT_TYPE	eEve;
        	DWORD_PTR	lParam; //Object Address
        	DWORD_PTR	wParam; //Group Type
        };
        ```

        - 여기서 DWORD가 아닌 DWORD_PTR을 선언하는 이유는 운영체제에 맞는 주소 크기만큼 알아서 메모리가 할당 되기 때문이다.
- 이벤트 함수 작성
    - 생성함수
        - 자주 호출될 기능이기 때문에 따로 함수용 헤더를 만들어 구현한다.

        ```cpp
        void CreateObject(CObject* _pObj, GROUP_TYPE _eGroup);
        ```

    - 이벤트 매니저 처리함수
        - 이벤트를 한번에 처리한다는 개념이 중요

        ```cpp
        void CEventMgr::update()
        {
        	for (size_t i = 0; i < m_eEvents.size(); ++i)
        	{
        		Excute(m_eEvents[i]);
        	}
        	m_eEvents.clear();
        }

        void CEventMgr::Excute(const tEvent& _eve)
        {
        	switch (_eve.eEve)
        	{
        	case EVENT_TYPE::CREATE_OBJECT:
        		// lParam : Object Address
        		// wParam : Group Type

        		CObject* pNewObj = (CObject*)_eve.lParam;
        		GROUP_TYPE eGroupType = (GROUP_TYPE)_eve.wParam;

        		CSceneMgr::GetInst()->GetCurScene()->AddObject(pNewObj, eGroupType);
        		break;
        	case EVENT_TYPE::DELETE_OBJECT:
        		break;
        	case EVENT_TYPE::SCENE_CHANGE:
        		break;
        	default:
        		break;
        	}
        }

        void CEventMgr::AddEvent(const tEvent& _eve)
        {
        	m_eEvents.push_back(_eve);
        }
        ```

    ### 삭제 이벤트 처리

    ---

    - 삭제 이벤트는 매우 처리가 복잡하다 왜냐하면, 오브젝트를 막 삭제하면 연관된 오브젝트들이 오류를 발생시킬 수 있기 때문이다.
    - 다른 오브젝트에서는 주소만으로 그 오브젝트가 삭제되었는지 아닌지 판단할 수 없기 때문이다.
    - 삭제 이벤트 처리 순서
        1. 삭제할 오브젝트를 정한다.
        2. 오브젝트는 바로 삭제되는게 아닌 삭제트리거를  true값으로 변경한다.
        3. 1프레임 업데이트 과정을 수행한다.
        4. 다음 프레임, 씬에서 업데이트를 할때 업데이트 처리를 하지 않는다.
        5. final update는 처리하고 rendering을 수행할때 씬에서 제거한다.
            - 여기서 충돌체 체크에대한 점검을 다시 수행한다.
        6. 이벤트처리에서 삭제 트리거가 true인 오브젝트를 모두 한번에 삭제한다.
    - 업데이트를 안하는 이유
        - 어차피 삭제 될 오브젝트 이기때문에 업데이트가 필요없고, 삭제트리거가 켜져있기 때문에 다른 오브젝트들과의 연결이 끊어진다.
    - 충돌 이벤트 처리
        - 계속 충돌중일때
            - 삭제트리거가 켜져있을땐 충돌을 해제해준다
        - 이제 충돌이 시작할때
            - 삭제트리거가 켜져 있다면 충돌을 시키지 않고 삭제한다.
        - 충돌이 끝난 상태이거나 충돌중이 아닐때
            - 이경우는 충돌에 관계가 없으므로 별도의 처리가 필요없다.