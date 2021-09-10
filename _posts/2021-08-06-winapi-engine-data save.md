---
layout: post
title:  "[C++][WinAPI] Create Game Engine -15 Save Data"
summary: "winapi game engine"
author: moon
date: '2021-08-06 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/savedata
usemathjax: true
---
## 그림판의 데이터 저장

---

- 블렌더, 그림판, 포토샵등을 보면 진행중인 작업을 저장해 나중에 다시 작업을 진행할 수 있다.
- 게임에서도 맵데이터 , UI데이터 등을 시스템메모리(RAM)이아니라 저장공간에 저장 할 수 있도록 설계해야한다.
- 시스템메모리는 컴퓨터가 꺼지면 모든 데이터가 날아가기 때문에 진행중인 작업을 이어서 하고 싶다면 데이터를 저장장치에 저장할 수 있도록 설계를 해야한다.

## 툴씬의 데이터 저장

---

- 기본적으로 맵에 대해서 저장하는 방법을 진행해보자
- 저장은 툴씬에서 하는거지만, 로딩은 씬에서 한다.
- 왜냐하면 씬에서 배경을 담당하는것이 맵 데이터이기 때문이다.
- 확장자는 파일이 어떤 형식을 가지고있는지 알려주는 글자이다, 확장자를 임의로 변경된다고 해서 파일의 구조가 변경되지 않는다.

### 파일 입출력

---

- 파일을 저장하기 위해서는 몇가지 조건을 만족해야한다.
    1. 데이터를 저장할 새로운 파일이 생겨야한다.
    2. RAM과 저장관리를 연결할 Stream이 생겨야한다.
- C++에서는 파일입출력 라이브러리를 지원한다.

    ```cpp
    //읽기는 각 fwrite를 fread로 바꾸어준다.
    SaveTile(const wstring& _relativePath)
    {
    	//커널 오브젝트 사용자가 다룰수 있는 범위 밖
    	//메모리간의 연결을 잡아주는역할
    	FILE* file = nullptr;
    	//wide string용 파일 입력 함수
    	_wfopen_s(&file, 
    						//파일의 경로(절대 경로), 
    						//사용방법(읽기'r',쓰기'w', 바이너리로 읽고쓸때는 b를 추가한다.),
    						);
    	//파일 열기 실패
    	assert(file);
    	//타일의 행과열의 개수 저장
    	UINT xCount = GetTileX();
    	UINT yCount = GetTileY();
    	//파일 쓰기
    	fwrite(&xCount, sizeof(UINT),1, file); 

    	//개별적으로 필요한 데이터를 저장
    	//타일 그룹의 벡터를 가져와서 save함수를 호출

    	//파일 입출력 닫기
    	fclose(file);
    }
    ```

    - FILE 구조체는 메모리간의 연결인 스트림을 대변한다고 생각, 그렇기 때문에 사용후에는 꼭 연결을 끊어줘야한다.
    - 쓰기 모드일때는 써야할 파일이 위치에 없으면 생성 시킨다.
    - 이어쓰기 옵션을 추가안하면 파일의 내용을 덮어쓰기 해버린다.
    - Tile의 정보는 씬에서 할게 아니라 Tile 클래스에서 결정을해서 입력을 해줘야한다.
    - 이건 모든 오브젝트에 해당되므로, 오브젝트 클래스에 설정해줘도 될것같다.

## Window Event로 파일 선택창 만들기

---

- 파일 저장 Diaglog는 기본적으로 windows에서 제공한다.
- 이러한 저장방법은 예시가 많이 나와있으니 외우지 말고, 참조를하여 자신에게 필요한 기능만 만들도록 하자

```cpp
OPENFILENAME ofn = {};

//이 구조체를 채우는건 documentation 보면서 한번 채워보자, 그게 더 좋다

//파일선태창을 열어내는 함수
//ture - save
//false - cancle
//당연히 입출려과 같이 불러오는건 이름만 다르다
if(GetSaveFileName(&ofn))
{
	//이곳에서 진짜 저장함수로 진행
	//GetSaveFileName만 제공, 
}
```