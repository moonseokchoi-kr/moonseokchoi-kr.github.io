---
layout: post
title:  "[C++][WinAPI] Create Game Engine -7 Collision"
summary: "winapi game engine"
author: moon
date: '2021-07-27 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/collision
usemathjax: true
---
## 충돌이 발생할때

---

- 단순히 오브젝트가 부딪혔을때만 충돌을 체크하면되는껄까?
- 정답은 NO이다. 충돌이 계속 유지 될수도 있고, 충돌시 여러 이벤트가 복합적으로 발생할 수도 있기때문에 이러한 이벤트들에 대한 체크도 필수적이다.
    - 만약 이러한 처리를 하지 않을시 프레임에 따라 데미지가 다르게 들어가는 치명적인 오류가 발생할 수도 있다.
- 모든 오브젝트에 충돌이 발생해야할까?
    - NO! Trigger나 UI같은경우 충돌을 감지하면 안되고 통과를 해야한다.

## 충돌클래스 설계

---

- 상속을 기반으로 할경우 확정성이 너무 떨어진다.
- 이를 위해 컴포넌트기반 구조를 사용한다.

    ## 컴포넌트 기반구조

    ---

    - 필요할 수 있는 멤버들을 모두 가지고 있되, 필요할때만 활성화를 시키는 구조를 말한다.
    - 상속이 아닌 객체를 위임받는 형태로 제작되기 때문에, 필요할때만 위임을 받아 사용하면 된다.
    - 이를 통해 훨씬 좋은 확장성을 확보할 수 있다.

- 충돌 클래스는 Object를 따라야하기 때문에 Object의 위치를 가져오기위해 전방선언이 필요하다.
- 기존의 Update방식과 달리, 프로그래머가 신경쓰지 않더라도 업데이트가 자동으로 발생하도록 해주는 부분이 필요하다(Scene finalupdate함수 추가)
- finalupdate는 이미 해야할 일이 정해져 있기 때문에 오버라이드를 금지한다.

    ```cpp
    //재정의 금지
    virtual void finalupdate() final;
    ```

- 함수의 실행 순서는 update→finalupdate→render이기 때문에 항상 위치가 옮겨진뒤 렌더링 된다.
- Collider의 경우 Offset을 가져야하는데 그이유는 플레이어나 오브젝트가 여러 충돌체를 작게나누어 가질수도 있기때문이다. 그렇기 때문에 원위치(플레이어위치)로부터인 상대적위치를 가져야한다.
- 최종위치는 위 두위치를 더하여 산출한다.
- 충돌체의 크기는 직접 설정할 수 있도록 해준다.

## 충돌 클래스 렌더링

---

- 렌더링방식에는 크게 두가지가 있다.
    - 선을이용해 빈공간을 만드는 방법
    - 사각형의 내부를 Hollow Brush로 채우는 방법
- 여기서는 사각형을 이용한 방법을 사욯한다.
- 이를 위해서 자주사용하는 BRUSH를 미리 설정해둔다. enum class를 통해 사용할 펜의 타입을 정의하고, 배열을 통해 저장한다.
- 이후 상속을 이용하여 필요할때만 렌더링을 이용할 수 있도록 설정한다.

## 충돌 판정

---

- 충돌판정은 앞써 설정한 그룹을 이용한다.
- 만약 그룹을 이용하지 않으면 씬안에 있는 모든 오보젝트에 대해 항상 충돌을 하고 있는지에 대한 체크를 해야해서 엄청난 연산을 소모하게된다.
- 이러한 충돌을 전담하는 함수는 따로 제작한다.
- 충돌을 판정하는 시점
    - final update이후에 처리를한다. 왜냐하면 update과정을 거치면서 충돌에 대한 결과가 변경이 될 수 있기때문에 모든 업데이트가 마무리 된뒤 충돌 판정을 처리한다.
    - 충돌 판정이 끝난뒤 렌더링을 진행한다.(간혹 프레임이 떨어지면 데미지계산이 먼저되고 물체가 맞는걸 볼 수 있는 이유중 하나)
- 충돌을 체크하는 법
    - 그룹 타입의 크기를 가로세로로 가지는 정사각형의 표가 있다고 가정하자

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c904bb64-249a-44f8-83b3-39a1bda8ba93/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c904bb64-249a-44f8-83b3-39a1bda8ba93/Untitled.png)

        그림으로 표현한다면 위와 같은 느낌일것이다.

    - 여기서 생각을 해보면 첫행과 첫열의 값이 반복되는 걸 알수 있다.
    - 그리고 사용자끼리의 충돌이나 몬스터끼리의 충돌을 위해 같은 그룹의 체크도 필요한다고 가정했을때 저 표에서 우리가 사용할 영역은

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69498e20-04ef-4c48-8d22-311d1c8ff118/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69498e20-04ef-4c48-8d22-311d1c8ff118/Untitled.png)

        노란색으로 칠한 영역이된다.

    - 그룹번호 중에 작은 것을 행으로 하고 큰것을 열로 하여 체크를 하면 손쉽게 가능하다.
    - 이후 비트연산을 통해 값을 추출한다.

    ```cpp
    CheckGroup(leftType, rightType)
    {
    	//항상 작은값을 행으로
    	UINT iRow = leftType
    	UINT iCol = rightType;
    	if(iCol<iRoW)
    	{
    			UINT iRow = rightType
    			UINT iCol = leftType;	
    	}
    	//해당하는 상태에 체크
    	if(m_arrCheck[iRow] & (1<<iCol))
    	{
    		m_arrCheck[iRow] &= ~(1<<iCol);
    	}
    	else
    	{
    		m_arrCheck[iRow] |=(1<<iCol);
    	}
    	
    }
    ```

## 충돌매니저

---

- 충돌의 설정까지 완료 했다면 이제 실제 충돌인지 검사를 해야한다.

```cpp
void CCollisionMgr::update()
{
//무슨 충돌을 체크할건지 검사
	for (UINT iRow = 0; iRow < (UINT)GROUP_TYPE::END; ++iRow)
	{
		
		for (UINT iCol = iRow; iCol < (UINT)GROUP_TYPE::END; ++iCol)
		{
			if (m_arrCheck[iRow] & (1 << iCol))
			{
				CollisionUpdateWithGroup((GROUP_TYPE)iRow, (GROUP_TYPE)iCol);
			}
		}
	}
}
```

```cpp
void CCollisionMgr::CollisionUpdateWithGroup(GROUP_TYPE _eleft, GROUP_TYPE _eRight)
{
	CScene* pCurScene = CSceneMgr::GetInst()->GetCurScene();

	const vector<CObject*>& vecLeft = pCurScene->GetObjectsWithType(_eleft);
	const vector<CObject*>& vecRight = pCurScene->GetObjectsWithType(_eRight);

	for (size_t i = 0; i < vecLeft.size(); ++i)
	{
//콜라이더가 있는지 체크
		if (nullptr == vecLeft[i]->GetCollider())
		{
			continue;
		}
		for (size_t j = 0; j < vecRight.size(); ++j)
		{
//콜라이더가 있는지, 자기자신이 아닌지 체크
			if (nullptr== vecRight[i]->GetCollider() || vecRight[i] == vecLeft[i])
			{
				continue;
			}
//실제 충돌처리
		}
	}
}
```

- 이제 실제 충돌처리할때 문제가 있다. 게임내 충돌체가 100개라면 충돌체끼리 충돌할 수 있는 경우의 수가 약 5000개가 나온다.
- 이러한 충돌체들의 충돌경우수를 저장하고 빠르게 찾아와야하기 때문에 탐색속도가 빠른 자료구조를 사용해야한다.
- 가장 추천하는것은 Hash Table이지만, 다른것도 괜찮다(고유한 ID를 따로 만들어주면 쉽다)
    - 복사를 통해 객체를 생성할경우도 ID가 고유해야하기때문에 깊은 복사를 해야한다.
    - 복사를 통해 충돌체를 새로 생성할경우 소유주가 없는게 맞기때문에 복사생성자에선 소유주를 설정해주지 않는다.
- 충돌체의 경우 대입을 할 경우가 없기 때문에 delete를 이용해 대입연산자를 없애줘야한다.

### 매니저에서 이전 프레임 충돌정보  저장

---

- union 이란?
    - 구조체와 달리 가장큰 자료형의 크기를 기준으로 공간을 공유한다.
    - 호출시 어떤 멤버를 부르느냐에 따라 공간이 형태가 달라진다(float를 부르면 float, int를 부르면 int로 공간을 인식)
    - 즉  union이란 자료형은  주어진 공간을 여러 형태로 사용하고 싶을때 사용한다.
- 충돌정보 맵 생성
    - 위에서 이용한 union을 이용하여 고유한 ID를 만든다

        ```cpp
        union COLLIDER_ID
        {
        	struct 
        	{
        		UINT Left_Id;
        		UINT Rigth_Id;
        	};

        	ULONGLONG ID;
        };
        ```

        유니온의 특징으로 이제 맵의 키값은 ID가 된다.

        ```cpp
        CCollider* pLeftCollider = vecLeft[i]->GetCollider();
        CCollider* pRightCollider = vecRight[i]->GetCollider();
        COLLIDER_ID ID;
        ID.Left_Id = pLeftCollider->GetId();
        ID.Rigth_Id = pRightCollider->GetId();
        iter = m_mCollisionInfo.find(ID.ID);
        ```

        이렇게 ID를 생성하고 사용하면 된다.

- 충돌 이벤트 정리
    - 충돌시 발생할 수 있는 이벤트
        - 이전부터 계속 충돌하는 경우
        - 지금 충돌이 시작된경우
        - 충돌이 끝난경우
    - 위 3가지를 각각 이벤트함수로 제작하여 조건에 맞게 호출을 해준다
        - OnCollision()
        - OnCollisionEnter()
        - OnCollisionExit()
- 각 오브젝트에서 이벤트 호출
- 부모클래스에 위의 이벤트를 가상함수로 구현해두고 필요한 함수만 오버라이딩해서 구현한다.

## Collider의 복사

---

- 현재 콜라이터를 복사를 할경우 이전 콜라이더의 정보가 그대로 복사되어 중복된 콜라이더로 복사된다.
- 따라서 깊은 복사를 통해 콜라이더의 정보를 변경해서 생성할 수 있도록 해야한다.

## Collision의 활용

---

- 이벤트트리거는 눈에 보이지 않지만, 플레이어나 오브젝트가 트리거 들어가게 되면 특정 이벤트가 발생한다.
- 다양한 기믹을 제작할 수 있기때문에 많이 사용된다.