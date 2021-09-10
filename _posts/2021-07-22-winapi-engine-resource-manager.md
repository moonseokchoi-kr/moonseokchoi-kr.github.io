---
layout: post
title:  "[C++][WinAPI] Create Game Engine -5 resource, resource Manager"
summary: "winapi game engine"
author: moon
date: '2021-07-22 12:35:23 +0900'
category: WinAPI
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, winapi, ui, grapic,engine
permalink: /blog/cpp/winapi/engine/resourcemanager
usemathjax: true
---
### Resource?

---

- 게임내 사용되는 이미지, 사운드, 애니메이션등을 전체적으로 말한다.
- 주로 상대경로를 통해 불러온다.
    - 게임이 설치될 장소가 어디인지 모르기 때문에 절대경로를 사용할 수 없다.
    - 따라서 게임 디렉토리 구조내에 리소스가 포함되어있어야한다(게임의 용량이 큰이유)
- 프로그램내에서는 키값을 통해 관리된다.
- 이러한 공통점을 모든 리소스들의 부모클래스로 사용될 클래스로 추상화한다.

```cpp
class CResource
{
public:
	CResource();
	virtual ~CResource();
public:
	virtual bool Load(const wstring& _path) = 0;
	void SetKey(const wstring& _key) { m_resourceKey = _key; }
	void SetRelateivePath(const wstring& _path) { m_relativePath = _path; }

	const wstring& GetKey() { return m_resourceKey; }
	const wstring& GetRelativePath() { return m_relativePath; }
private:
	wstring m_resourceKey;
	wstring m_relativePath;
};
```

## Directory Struct

---

- 리소스를 관리하기 위해서는 경로를 명확히 정해야한다.
- 특히 개발할때와 실제 실행하는 환경이 다르기 때문에 이부분을 신경써야한다.
- 따라서 앞으로 게임이 설치되어 배포할 폴더에 저장을 할 수 있도록 구조를 잡아야한다.

```cpp
class CResource
{
	public:
		CResource();
		virtual ~CResource();
	public:
		virtual bool Load() = 0;
	
	private:
		w
}
```

## Path Manager

---

- Diretory Struct를 잡았다면 이제 확정된 구조에서 Path를 관리할 매니저가 필요
- 매니저가 있다면 Resource명만 알면 어디서든지 리소스를 이용할 수 있게된다.
- 이를 위해서 현재 실행되고 있는 디렉토리가 어디인지 파악할 필요가 있고. 이는 기본제공함수로 해결할수 있다.

    ```cpp
    void CPathMgr::init()
    {
    	//경로는 255글자 제한
    	GetCurrentDirectory(255, m_contentPath);
    	
    }

    ```

- 이때 발생하는 문제는 Visual Studio에서 실행될때이다, 우리가 설정한 구조가 아닌 visual studio에서 설정한 환경에서 구동되다보니 directory가 달라지게 된다.
- 이럴때는 속성-디버깅에 들어가 작업디렉토리를 우리가 설정한 디렉토리구조로 변경해주면된다.
- 문제를 해결했으니, 다시 디렉토리 경로를 관리하는 코드를 작성하자

    ```cpp
    void CPathMgr::init()
    {
    	//경로는 255글자 제한
    	GetCurrentDirectory(255, m_contentPath);
    	
    	//실제에서는 함수를 쓰지만 우리는 학생이니까 상위폴더 경로를 가져오는
    	//함수를 만들어보자

    	int iLen = wcsLen(m_contentPath);
    	
    //상위폴더로 가자
    	for(int i=iLen-1; i<=0; --i)
    	{
    		if('\\' == m_contentPath[i])
    		{
    			m_contentPath = '\0';
    			break;
    		}
    	}

    	wcscat_s(m_contentPath, 255, L"\\bin\\content\\");

    ```

## Texture

---

- 2D게임에서는 사용되는 이미지를 말하며 3D게임에서는 주로 입체모델의 표면을 표시하는 그림들을 말한다.
- 이 이미지를 적용하기 위해서는 Device Context가 Texture마다 필요하다.
- 로드한 이미지는 BitMap으로 저장하고 그 BitMap을 DeviceContext에 넣어 게임내에 표현하게 된다

```cpp
class CTexture :
    public CResource
{
public:
    CTexture();
    ~CTexture();
public:
    // CResource을(를) 통해 상속됨
    virtual bool Load(const wstring& _path) override;

    UINT Width() { return m_bitMapInfo.bmWidth; }
    UINT Height() { return m_bitMapInfo.bmHeight; }

    HDC GetDC() { return m_hdc; }
private:
    HBITMAP m_bitMap;
    HDC m_hdc;
    BITMAP m_bitMapInfo;
};
```

- Texture내에는 원본이미지 자체가 직사각형이기 때문에 배경색을 필수적으로 가지게된다.
- 그렇기 때문에 Magenta색상을 배경에 적용하고, 조건체크를 통해 배경을 날려버린다.
- 이때 사용하는 함수가 TransparentBlt()이다

    [TransparentBlt function (wingdi.h) - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/api/wingdi/nf-wingdi-transparentblt)

- 이 함수는 구현부가 lib파일에 저장되어있기때문에 사용시 라이브러리 참조를한다.

    ```cpp
    #pragma comment(lib, "Msimg32.lib")
    ```

## ResourceManeger

---

- 리소의 키와 texture를 둘다 같이 관리할 수 있는 Map 자료구조를 통해 관리한다.
- 리소스 매니저를 통해 필요한 리소스를 한번만 로딩하고 이후 필요할때 자료구조에서 불러 사용한다.
- 매니저를 통해 자원을 할당하고 자원을 해제한다.