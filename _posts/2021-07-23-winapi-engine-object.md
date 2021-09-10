---
layout: post
title:  "[C++][WinAPI] Create Game Engine -6 Object"
summary: "winapi game engine"
author: moon
date: '2021-07-24 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/object
usemathjax: true
---
## Object Class

---

- 장면에 존재하는 벽, 몬스터, 플레이어등의 부모가 되는 클래스
- 프레임별로 해야할일을 담당하는 update와 장면을 그리는 render함수가 존재
- 이때 render의 경우 모두 같은 방식을 사용하지만 update는 객체마다 다르므로 순수가상함수로 구현되어야 한다.

    ```cpp
    class CObject
    {
    private:
    	Vec2	m_vPos;
    	Vec2    m_vScale;

    public:
    	void SetPos(Vec2 _vPos) { m_vPos = _vPos; }
    	void SetScale(Vec2 _vScale) { m_vScale = _vScale; }

    	Vec2 GetPos() { return m_vPos; }
    	Vec2 GetScale() { return m_vScale; }

    public:
    	virtual void update() = 0;
    	virtual void render(HDC _dc);

    public:
    	CObject();
    	virtual ~CObject();
    };
    ```

## Object Class 개선 - 1

---

- 지금 ObjectClass를 복사한다면 어떻게 될까?
    - 충돌체를 가지고 있는 객체에 한해, 충돌체의 주소가 같은것으로 설정되는 문제점이 있다.
    - 즉 새로운 복사 생성자를 생성해서 깊은 복사를 시켜줘야한다.
- 깊은 복사 생성자

    ```cpp
    CObject::CObject(const CObject& _origin)
    	:m_strName(_origin.m_strName)
    	,m_vPos(_origin.m_vPos)
    	,m_vScale(_origin.m_vScale)
    	,m_pCollider(nullptr)
    	,m_bAlive(true)
    {
    	m_pCollider = new CColider(_origin.m_pCollider);
    	m_pCollider->m_pOwner= this;
    }
    ```

    - 콜라이더와 삭제유뮤에 대해서만 변경을 해주면 된다.

- 이 깊은 생성자를 호출하는데 문제는 바로 호출이 복잡하다

    ```cpp
    CObject* player = new CPlayer;

    new CPlayer(*(CPlayer*) player);
    ```

- 이러한 문제를 해결하기 위해 clone함수를 구현한다

    ```cpp
    CPlayer* Clone() {return new CPlayer(*this);}
    ```