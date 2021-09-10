---
layout: post
title:  "[C++][WinAPI] Create Game Engine -4 Scene, Scene Manager"
summary: "winapi game engine"
author: moon
date: '2021-07-21 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/scene
usemathjax: true
---
## Scene

---

- 게임의 스테이지를 뜻하기도 하지만, 메인화면이나 UI창등이 모인집합도 Scene이다
- 즉, 게임의 오브젝트들의 집합을 저장하는곳
- 이러한 장면에는 여러타입의 오브젝트들이 저장되기 때문에 오브젝트 타입별로 저장할 필요가 있다.
- 씬 필수 함수의 역할
    - Enter()
        - 씬에 진입할때 실행되는 함수로 씬의 초기화를 담당
    - Start()
        - 씬에 등록된 오브젝트들이 업데이트 전에 설정해야할 값들을 설정하는 곳
    - Exit()
        - 씬이 종료되면서 처리해야할 일을 담당한다.
- 기본적으로 상속 구조를 이용하기 때문에 모든 씬에서 이용할 기능은 최상위 부모 클래스에 만들어두는게 좋다
- 씬 클래스 작성예시

```cpp
CScene::CScene()
	:m_iTileX(0)
	,m_iTileY(0)
{
}

CScene::~CScene()
{
	for (UINT i = 0; i < (UINT)GROUP_TYPE::END; ++i)
	{
		for (size_t j = 0; j < m_arrObj[i].size(); ++j)
		{
			// m_arrObj[i] 그룹 벡터의 j 물체 삭제
			delete m_arrObj[i][j];
		}
	}
}

void CScene::start()
{
	for (UINT i = 0; i < (UINT)GROUP_TYPE::END; ++i)
	{
		for (size_t j = 0; j < m_arrObj[i].size(); ++j)
		{
			if (!m_arrObj[i][j]->IsDead())
			{
				m_arrObj[i][j]->start();
			}
		}
	}
}

void CScene::update()
{
	for (UINT i = 0; i < (UINT)GROUP_TYPE::END; ++i)
	{
		for (size_t j = 0; j < m_arrObj[i].size(); ++j)
		{
			if (!m_arrObj[i][j]->IsDead())
			{
				m_arrObj[i][j]->update();
			}			
		}
	}
}

void CScene::finalupdate()
{
	for (UINT i = 0; i < (UINT)GROUP_TYPE::END; ++i)
	{
		for (size_t j = 0; j < m_arrObj[i].size(); ++j)
		{
			m_arrObj[i][j]->finalupdate();
		}
	}
}

void CScene::render(HDC _dc)
{
	for (UINT i = 0; i < (UINT)GROUP_TYPE::END; ++i)
	{
		if ((UINT)GROUP_TYPE::TILE == i)
		{
			render_tile(_dc);
			continue;
		}
		vector<CObject*>::iterator iter = m_arrObj[i].begin();

		for (; iter != m_arrObj[i].end();)
		{
			if (!(*iter)->IsDead())
			{
				(*iter)->render(_dc);
				++iter;
			}	
			else
			{
				iter = m_arrObj[i].erase(iter);
			}
		}
	}
}

void CScene::render_tile(HDC _dc)
{
	//카메라가 보고있는 기준으로 카메라 범위에만있는 타일을 렌더하기 위함
	//성능의 향상으로 이어짐

	const vector<CObject*>& vecTile = GetGroupObject(GROUP_TYPE::TILE);

	Vec2 vCamLook = CCamera::GetInst()->GetLookAt();
	Vec2 vResolution = CCore::GetInst()->GetResolution();
	//카메라 좌상단의 위치
	Vec2 vLeftTop = vCamLook - vResolution / 2.f;

	int iTileSize = TILE_SIZE;

	int iLTCol = (int)vLeftTop.x / iTileSize;
	int iLTRow = (int)vLeftTop.y / iTileSize;

	

	int iCamWidthTileCount = ((int)vResolution.x / iTileSize)+1;
	int iCamHeightTileCount = ((int)vResolution.y / iTileSize)+1;

	 

	for (int iCurRow = iLTRow; iCurRow < (iLTRow + iCamHeightTileCount); ++iCurRow)
	{
		for (int iCurCol = iLTCol; iCurCol < (iLTCol + iCamWidthTileCount); ++iCurCol)
		{
			//인덱스가 음수이거나, 열이 음수이면 카메라가 렌더링하면 안된다.
			if(iCurCol<0 || m_iTileX <=iCurCol || iCurRow<0||m_iTileY <= iCurRow)
				continue;

			//현재 인덱스
			int iIdx = m_iTileX * iCurRow + iCurCol;

			vecTile[iIdx]->render(_dc);
		}
	}

}

void CScene::DeleteGroup(GROUP_TYPE _eTarget)
{
	Safe_Delete_Vec(m_arrObj[(UINT)_eTarget]);
}

void CScene::DeleteAll()
{
	for (UINT i = 0; i < (UINT)GROUP_TYPE::END; ++i)
	{
		DeleteGroup((GROUP_TYPE)i);
	}
}

void CScene::CreateTile(UINT _iXCount, UINT _iYCount)
{
	DeleteGroup(GROUP_TYPE::TILE);
	CTexture* tile = CResMgr::GetInst()->LoadTexture(L"Tile", L"texture\\tile\\TILE.bmp");
	m_iTileX = _iXCount;
	m_iTileY = _iYCount;
	for (int i = 0; i < _iYCount; ++i)
	{
		for (int j = 0; j < _iXCount; ++j)
		{
			CTile* pTile = new CTile;
			pTile->SetPos(Vec2((float)(j * TILE_SIZE), (float)(i * TILE_SIZE)));
			pTile->SetTexture(tile);
			AddObject(pTile, GROUP_TYPE::TILE);
		}
	}
}

void CScene::LoadTile(const wstring& _strRelativeData)
{
	FILE* file = nullptr;
	wstring strPath = CPathMgr::GetInst()->GetContentPath();
	strPath += _strRelativeData;
	//파일 열기
	_wfopen_s(&file, strPath.c_str(), L"rb");

	assert(file);
	//파일 읽기
	UINT xCount=0;
	UINT yCount=0;

	fread(&xCount, sizeof(UINT), 1, file);
	fread(&yCount, sizeof(UINT), 1, file);

	CreateTile(xCount, yCount);

	const vector<CObject*>& vecTile = GetGroupObject(GROUP_TYPE::TILE);
	for (size_t i = 0; i < vecTile.size(); ++i)
	{
		((CTile*)vecTile[i])->Load(file);
	}

	fclose(file);

}
```

## SceneManager

---

- 게임의 모든 장면을 관리하는 클래스
- 장면을 타입별로 저장하여 관리한다.
- 이를 통해 장면이 전환되었을때의 렌더링이나, 업데이트등을 수행할 수 있도록한다.
- 자료구조에 게임내 존재하는 모든 씬을 등록하고 관리하게 된다.
- 현재씬을 가져 오고 싶을때 매니저 클래스를 이용해야한다.