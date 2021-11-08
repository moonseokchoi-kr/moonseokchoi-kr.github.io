---
layout: post
title:  "[C++][Direct2D] Create Game Engine -8 object, component"
summary: "Direct2D game engine"
author: moon
date: '2021-09-14 12:35:23 +0900'
category: Direct2D
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, Direct2D, ui, grapic,engine
permalink: /blog/cpp/Direct2D/engine/object-component
usemathjax: true
---
## 게임엔진 구조 설계 - Object설계

---

- 여러 물체들을 그릴 수록 있도록 엔진구조를 설계 해야한다.
- 각 위치를 잡고, 필요한 리소스들을 가져와 렌더링이 될 수 있도록 해야한다.
- 완전한 컴포넌트 기반 설계를 중심으로 만든다.
- 필요한 컴포넌트를 붙일 수 있도록 설계한다.
- 컴포넌트를 어떻게 추가하느냐에 따라 오브젝트의 역할을 정할 수 있다.
    
    예를 들어 카메라 컴포넌트를 추가하면 카메라 역할을 하게 되고, 빛 컴포넌트르 넣어주면 조명의 역할을 하게된다.
    
- 전체 코드
    
    ```cpp
    #include "pch.h"
    #include "CGameObject.h"
    #include "CComponent.h"
    #include "CMeshRender.h"
    CGameObject::CGameObject()
    	:m_arrComponent{}
    {
    }
    
    CGameObject::~CGameObject()
    {
    	Safe_Delete_Array(m_arrComponent);
    }
    
    void CGameObject::update()
    {
    	for (UINT i = 0; i < (UINT)COMPONENT_TYPE::END; ++i)
    	{
    		if (nullptr != m_arrComponent[i])
    		{
    			m_arrComponent[i]->update();
    		}
    	}
    }
    
    void CGameObject::render()
    {
    	if (nullptr == MeshRender())
    		return;
    	MeshRender()->render();
    }
    
    void CGameObject::UpdateData()
    {
    }
    
    void CGameObject::AddComponent(CComponent* _pCom)
    {
    	UINT iType = (UINT)_pCom->GetType();
    	assert(!m_arrComponent[iType]);
    	_pCom->m_pOwner = this;
    	m_arrComponent[iType] = _pCom; 
    }
    ```
    
    ```cpp
    class CGameObject :
        public CEntity
    {
    private:
        CComponent* m_arrComponent[(UINT)COMPONENT_TYPE::END];
    
    public:
        void update();
        void render();
    
    public:
        void AddComponent(CComponent* _pCom);
        CComponent* GetCompnent(COMPONENT_TYPE _eType) { return m_arrComponent[(UINT)_eType]; }
    
        GET_COMPONENT(Transform, COMPONENT_TYPE::TRANSFORM);//매크로정의 아래 참조
        GET_COMPONENT(MeshRender, COMPONENT_TYPE::MESHRENDER);
    public:
        virtual void UpdateData();
    public:
        CGameObject();
        ~CGameObject();
    ```
    
- 컴포넌트 가져오기
    - 컴포넌트를 가져오는 함수를 만들게 되면 다음과 같다
        
        ```cpp
        CComponent* CGameObject::GetComponent(COMPENENT_TYPE _type)
        {
        	return m_arrComp[(UINT)_type];
        }
        ```
        
    - 이렇게 반환하게되면 매번 캐스팅을 해서 사용해야하는 문제점이있다.
    - 따라서 매크로를 통해 가져오는 함수를 만들어 두고, 컴포넌트를 추가할때 매크로로 함수를 선언해주면 훨씬구조가 간단해진다.
        - 매크로 선언 예
        
        ```cpp
        #define GET_COMPONENT(ComponentName, EnumName)C##ComponentName* ComponentName(){return (C##ComponentName*)m_arrComponent[(UINT)EnumName];}
        ```
        

### 컴포넌트 설계

---

```cpp
class CComponent: public CEntity
{
		//모든 컴포넌트의 최상위 클래스
	private:
			const COMPOENET_TYPE m_type;//자신이 어떤 컴포넌트인지 기억하는 목적,선언후 변경불가
	public:
			virtual void update(){}
			COMPONENT_TYPE GETType(){return m_type;}
	public:
			CComponent(COMPONENT_TYPE _type);
			~CComponent();
}
```

- 이렇게 설정하는 이유는 한번 설정된 컴포넌트의 타입을 변경하지 못하도록 하기 위해서이다.
- 또한 새로운 컴포넌트 생성시 타입설정을 안하는 경우를 방지한다.

### Transform

---

- 물체의 이동, 크기변경, 회전에 관한 업데이트를 담당 한다.
- 물체의 위치, 크키, 각도의 대한 정보를 관리하는 컴포넌트이다.
    
    ```cpp
    #pragma once
    #include "CComponent.h"
    class CTransform :
        public CComponent
    {
    private:
        Vec3 m_vPos;
        Vec3 m_vScale;
        Vec3 m_vRot;
    public:
        void SetPos(Vec3 _vPos) { m_vPos = _vPos; }
        void SetScale(Vec3 _vScale) { m_vScale = _vScale; }
        void SetRot(Vec3 _vRot) { m_vRot = _vRot; }
    
        Vec3 GetPos() { return m_vPos; }
        Vec3 GetScale() { return m_vScale; }
        Vec3 GetRot() { return m_vRot; }
    public:
        virtual void update();
        virtual void UpdateData();
    public:
        CTransform();
        ~CTransform();
    };
    ```
    
    ```cpp
    #include "pch.h"
    #include "CTransform.h"
    #include "CConstBuffer.h"
    
    #include "CKeyMgr.h"
    #include "CTimeMgr.h"
    #include "CDevice.h"
    CTransform::CTransform()
    	:CComponent(COMPONENT_TYPE::TRANSFORM)
    {
    }
    
    CTransform::~CTransform()
    {
    }
    
    void CTransform::update()
    {
    	// 키 입력에 따른 움직임
    	if (KEY_HOLD(KEY::W))
    	{
    		m_vPos.y += fDT * 0.5f;
    	}
    	if (KEY_HOLD(KEY::S))
    	{
    		m_vPos.y -= fDT * 0.5f;
    	}
    	if (KEY_HOLD(KEY::A))
    	{
    		m_vPos.x -= fDT * 0.5f;
    	}
    	if (KEY_HOLD(KEY::D))
    	{
    		m_vPos.x += fDT * 0.5f;
    	}
    }
    
    void CTransform::UpdateData()
    {
    	/// <summary>
    /// 상수버퍼는 지정한 셰이더에서만 사용할 수 있다. 만약 다른 셰이더에서 사용하고 싶을경우 추가적으로 지정할 필요가 있다.
    /// </summary>
    	CConstBuffer* cb = CDevice::GetInst()->GetConstBuffer(CB_TYPE::TRANSFORM);
    	cb->SetData(&m_vPos, sizeof(Vec4));
    	cb->SetPipelineState(PS_VERTEX);
    	cb->UpdateData();
    }
    ```
    

### MeshRender

---

- 물체의 렌더를 담당하는 클래스, 셰이터, 메쉬, 텍스처를 가지고 있으며 실질적인 렌더링을 담당한다.
- 정점데이터나 셰이더가 준비되지 않았을 경우 제대로 렌더링이 진행될 수 없기 때문에 문제를 발생시켜줘야한다.
- 셰이더에서 사용하는 상수버퍼들의 업데이트를 렌더링 직전에 실행시켜줘야한다. 정점에 대한 모든 처리가 끝난다음에 이 처리가 진행되어야하기 때문이다.
- 전체코드
    
    ```cpp
    #pragma once
    #include "CComponent.h"
    
    class CTexture;
    class CShader;
    class CMesh;
    class CMeshRender :
        public CComponent
    {
    private:
        CTexture* m_pTex;
        CShader* m_pShader;
        CMesh* m_pMesh;
    public:
        void SetTexture(CTexture* _pTex) { m_pTex = _pTex; }
        void SetShader(CShader* _pShader) { m_pShader = _pShader; }
        void SetMesh(CMesh* _pMesh) { m_pMesh = _pMesh; }
    public:
        virtual void update();
        void render();
        virtual void UpdateData();
    public:
        CMeshRender();
        ~CMeshRender();
    };
    ```
    
    ```cpp
    CMeshRender::CMeshRender()
    	:CComponent(COMPONENT_TYPE::MESHRENDER)
    	,m_pTex(nullptr)
    	,m_pMesh(nullptr)
    	,m_pShader(nullptr)
    {
    }
    
    CMeshRender::~CMeshRender()
    {
    
    }
    
    void CMeshRender::update()
    {
    }
    
    void CMeshRender::render()
    {
    	if (nullptr == m_pShader || nullptr == m_pMesh)
    	{
    		return;
    	}
    
    	//상수버퍼 업데이트
    	GetOwner()->Transform()->UpdateData();
    
    	//Shader 업데이트
    	m_pShader->UpdateData();
    
    	//텍스처 업데이트
    	if (nullptr != m_pTex)
    	{
    		m_pTex->SetPipelineState(PS_PIXEL, 0);
    		m_pTex->UpdateData();
    	}
    
    	//메쉬 렌더
    	m_pMesh->Render();
    }
    
    void CMeshRender::UpdateData()
    {
    }
    ```
    
    CMeshRender.cpp