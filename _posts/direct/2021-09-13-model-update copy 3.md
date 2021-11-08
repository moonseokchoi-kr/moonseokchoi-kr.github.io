---
layout: post
title:  "[C++][Direct2D] Create Game Engine -6 ResourceManager, Texture"
summary: "Direct2D game engine"
author: moon
date: '2021-09-13 12:35:23 +0900'
category: Direct2D
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, Direct2D, ui, grapic,engine
permalink: /blog/cpp/Direct2D/engine/texture
usemathjax: true
---
## Resource Manager

---

### 자원관리

---

- 이전의 WINAPI에서 ResourceManager를 설계했을때는 리소스의 양이 적어, 필요한 리소스들을 맵을 다 따로선언 관리했지만, 다이렉트부터는 매우 리소스양이 방대해지기 때문에 비효율적이다.
- 따라서 배열과 맵,  그리고 열거를 같이 이용해 관리하는것이 효율적이다.
    - 
    
    ```cpp
    //리소스 열거타입 선언
    enum class RES_TYPE
    {
    	//리소트타입 정의
    	END,
    }
    
    ///리소스 클래스내에 선언시
    array<unoredered_map<wstring, CRes*>, (UINT)RES_TYPE>::END> m_Res;
    
    ///리소스를 추가하거나 삭제하거나 찾아올 함수를 구현할 필요있음
    ///template 함수로 구현
    template<typename T>
    void AddRes(const wstring* _key, T* _res);
    
    template<typename T>
    T* FindRes(const wstring* _key);
    ```
    
    ### 리소스 추가하기
    
    ---
    
    ```cpp
    inline void CResMgr::AddRes(const wstring& _key, T* _res)
    {
    //클래스에는 고유한 ID를 가지고 있는데 이걸 이런식으로 가져올수 있음
    //이부분은 별도의 함수로 분리하여 갖고있는게 맞을듯
    //계속 중복해서 사용되는데 굳이 이렇게 사용할 이유는 없다.
    	const type_info& info = typeid(CMesh); 
    	const type_info& t = typeid(T)
    	RES_TYPE type = RES_TYPE::END;
    	//이렇게 검사를 해서 truer가 나오면 T가 CMesh라는 소리
    	if(t.hash_code() == info.hash_code())
    	{
    		type= RES_TYPE::MESH;
    	}
    
    	//위에서 찾은 타입과 포인터,키를 이용해 리소스를 추가한다
    	m_Res[(UINT)tpye].insert(make_pair(_key,_res));
    }
    ```
    
    ### 리소스 검색하기
    
    ---
    
    ```cpp
    inline T* CResMgr::FindRes(const wstring& _key)
    {
    //클래스에는 고유한 ID를 가지고 있는데 이걸 이런식으로 가져올수 있음
    //이부분은 별도의 함수로 분리하여 갖고있는게 맞을듯
    //계속 중복해서 사용되는데 굳이 이렇게 사용할 이유는 없다.
    	const type_info& info = typeid(CMesh); 
    	const type_info& t = typeid(T)
    	RES_TYPE type = RES_TYPE::END;
    	//이렇게 검사를 해서 truer가 나오면 T가 CMesh라는 소리
    	if(t.hash_code() == info.hash_code())
    	{
    		type= RES_TYPE::MESH;
    	}
    
    	//위에서 찾은 타입과 포인터,키를 이용해 리소스를 검색한다
    	unoredered_map<wstring, CRes*>::iterator iter == m_Res[(UINT)tpye].find(_key);
    	
    	if(iter == m_Res[(UINT)type)].end())
    	{
    			return null;
    	}
    }
    ```
    
    ### 리소스 불러오기
    
    ---
    
    ```cpp
    //CResource.h
    virtual int Load(const wstring& _strFilePath/*실제 경로*/) { assert(nullptr); return E_FAIL; }
    ```
    
    ```cpp
    template<typename T>
    inline Ptr<T> CResMgr::Load(const wstring& _key, const wstring& _strRelativePath)
    {
    	RES_TYPE eType = GetResType<T>();
    
    	map<wstring, CRes*>::iterator iter = m_Res[(UINT)eType].find(_key);
    
    	if (iter != m_Res[(UINT)eType].end())
    	{
    		return (T*)iter->second;
    	}
    
    	wstring strPath = CPathMgr::GetInst()->GetContentPath();
    	strPath += _strRelativePath;
    
    	CRes* pRes = new T;
    	if (FAILED(pRes->Load(strPath)))
    	{
    		MessageBox(nullptr, L"리소스 로딩 실패", L"리소스 로딩 에러", MB_OK);
    		assert(nullptr);
    	}
    
    	AddRes<T>(_key, (T*)pRes);
    
    	return Ptr<T>((T*)pRes);
    }
    ```
    
    ### 리소스 저장하기
    
    ---
    
    ```cpp
    //CResource.h
    virtual int Save(const wstring& _strRelativePath/*상대경로*/) { assert(nullptr); return E_FAIL; };
    ```
    

### 기본 리소스 생성

---

- 사용자가 기본적으로 이용할 수 있는 리소스를 제공 해야한다.
- 매쉬, 마테리얼, 셰이더, 사운드, 텍스쳐등 모두 제공이 되어야한다.
    
    ```cpp
    void CResMgr::CreateDefaultMesh(){}
    void CResMgr::CreateDefaultSound(){}
    void CResMgr::CreateDefaultShader(){}
    void CResMgr::CreateDefaultTexture(){}
    void CResMgr::CreateDefaultMatrial(){}
    ```
    

## Texture

---

- 이미지 데이터들도 버텍스나, 인덱스처럼 시스템과 GPU둘다 메모리에 가지고 있어야한다.
- 시스템메모리에있는 데이터를 GPU에 보내는 방법은 DirectX 에서 지원을 해주나 문제는, 하드디스크 메모리에 있는 이미지들을 불러올 방법을 기본적으로 제공을 해주지 않는다.
- 이러한 문제를 해결하기 위해 Micrsoft에서 별도의 프로젝트로 제공하고 있다. lib파일을 만드는 방법은 이전의 포스트를 참조바란다.
    
    [Direct2D - 1 [라이브러리]](https://www.notion.so/Direct2D-1-cbde3de73afb4d55a3e595529090136f)
    
    [GitHub - microsoft/DirectXTex: DirectXTex texture processing library](https://github.com/microsoft/DirectXTex)
    
- 모든 준비작업이 끝나면 이제 Texture 클래스를 작성해야한다.
- 파일에서 로딩한 이미지 데이터를 관리하는 클래스를 제공해준다(ScratchImage)
- 이 라이브러리에서 기본적으로 여러 확장자를 위한 별도의 로딩함수를 제공해주기 때문에 사용자가 작업해야하는건 로딩할 이미지 파일의 확장자를 구분해야하는것이다.
    
    ```cpp
    HRESULT hr = E_FALE;
    //파일경로에서 확장자 추출
    wchar_t arrExt[50] = L"";
    _wsplitpath_s(_strFilePath.c_str(),nullptr,0,nullptr,0,nullptr,0,arrExt,50);
    
    //dds
    if(!wcscmp(L".DDS",arrExt)||!wcscmp(L".dds",arrExt))
    {
    	hr= FAILED(LoadFromDDSFile(_strFilePath.c_str(), DDS_FLAGS_NONE, nullptr, m_image)))
    	
    }
    //tga
    else if(!wcscmp(L".tga",arrExt)||!wcscmp(L".TGA",arrExt))
    {
    	hr = FAILED(LoadFromTGAFile(_strFilePath.c_str(),nullptr,m_image));
    }
    //ICF(jpe,png,...)
    else
    {
    	hr = LoadFromWICFile(_strFilePath.c_str(),WIC_FLAGS_NONE,nullptr,m_image);
    }
    
    //실패했는지 검사
    if(FAILED(hr))
    	return E_FALE;
    
    //시스템 메모리에서 GPU메모리로 전달
    //Texture2D를 만들어서 전달
    
    //RTV,DSV와 같은 방식이나 사용목적이 다름
    //텍스처는 저둘의 출력하는 타겟의 데이터로 사용
    //결국 Texture이기 때문에 GPU에전달하려면 View에 연결해서 보내야하는데
    //이때 사용하는 View가 ShaderResourceView이다.
    
    //원래는 CDeivce에서 진행했던 과정을 진행
    //(텍스처 객체생성-> description작성->device로 texture메모리할당-> view생성)
    //이 모든 과정을 한방에 통합한 라이브러리도 제공해준다.
    //이 함수의 결과로 View바로 만들어지기 때문에 역으로 가져와야한다.
    CreateShaderResourceView(DEVICE, m_image.GetImages(), m_image.GetImageCount(), m_image.GetMetadata(), m_SRV.GetAddressOf());
    
    m_SRV->GetResource((ID3D11Resource**)m_tex2D.GetAddressOf());
    m_tex2D->GetDesc(&m_desc);
    
    return S_OK;
    
    ```
    
- 모델에 텍스처 적용
    - 셰이더 코드를 통해 모델에 텍스처를 적용할 수 있음
    - 이때 사용하는 레지스터가 텍스처 레지스터이다
        
        ```cpp
        //HLSL에 이렇게 선언한다,
        Texture2D tex : register(t0)
        ```
        
    - 상수버퍼와 같이 레지스터 번호로 필요한 텍스처를 선택해서 사용하게된다.
    
    ```cpp
    CTexture* pTex = CResMgr::GetInst()->FindRes<CTexture>(L"Background");
    pTex->SetPipelineStage(PIPELINE_STAGE::PS_PIXEL);
    pTex->SetRegisterNum(0);
    pTex->UpdateData();
    ```
    
    - 위의 코드에서 보이듯 픽셀셰이더는 적용할 스테이지와, 사용할 레지스터 번호를 지정해줘야한다.
    - 이 방법은 앞써 만든 상수버퍼와 같은 방식이다.
    - 그 다음 셰이더와 바인딩을 해줘야한다.
    
    ```cpp
    void UpdateData()
    {
    	CONTEXT->VSSetShaderResource(0, nullptr, m_SRV.GetAddressOf())
    }
    ```
    
    - 이제 부터는 셰이더에서 픽셀데이터를 매 픽셀마다 입히는 작업을 수행한다.
    - 픽셀이 텍스처를 통해 색을 질하는 방벙은 픽셀에 해당하는 좌표를 찾아 그곳의 색을 가져오게 된다.  이 과정을 샘플링이라고 한다.
        
        ```cpp
        samplerState sam : reagister(s0)
        ```
        
    - 이때 알아야하는 개념이 UV좌표계이다
    - UV좌표계는 좌상단을 (0,0) 우상단(1,0), 좌하단(0,1), 우하단(1,1) 이다.
    - 가운데 중심은 (0.5,0.5)이다.
    - 이 UV의 대한 좌표는 정점들이 가지고 있어야하는데 그래야만 각픽셀들이 UV의 좌표를 알 수 있다.(보간과정을 통해 알수 있음)
    
    ```cpp
    extern D3D11_INPUT_LAYOUT_DESC g_layout[3]
    ```
    
    - 정점에 데이터가 늘어났으므로 레이아웃도 추가로 작성해야한다.
- 샘플러 바인딩
    - 아래 코드를 보면 샘플러도 레지스터라는걸 알 수 있다.
    
    ```cpp
    samplerState sam : reagister(s0)
    ```
    
    - 이전의 레지스터에는 개발자가 데이터를 준비하여 버퍼를 연결해주는 식으로 사용되었다.
    - 하지만 sampler의 경우 Default가 있기 때문에 굳이 만들어 줄 필요는 없지만, DirectX에서는 이를 비정상적인 방식으로 보기 때문에 무수히 많은 오류를 표시해준다.
    - 샘플러는 2~3개 정도의 옵션을 설정해놓으면 모든 샘플링에 맞출수 있기 때문에 미리 생성을 다 만들어 놓은다음 바인딩을 걸어놓으면 된다.
    - 그렇기 때문에 Device초기화 때 같이 생성을 하고 프로그램 종료시까지 유지하면된다.
    
    ```cpp
    HRESULT CDevice::CreateSampler()
    {
    	D3D11_SAMPLER_DESC desc = {};
    	desc.AddressU = D3D11_TEXTURE_ADDRESS_MODE::D3D11_TEXTURE_ADDRESS_WRAP;
    	desc.AddressV = D3D11_TEXTURE_ADDRESS_MODE::D3D11_TEXTURE_ADDRESS_WRAP;
    	desc.AddressW = D3D11_TEXTURE_ADDRESS_MODE::D3D11_TEXTURE_ADDRESS_WRAP;
    	desc.Filter = D3D11_FILTER::D3D11_FILTER_ANISOTROPIC;//이방선필터링
    
    	DEVICE-> CreateSamplerState(&desc, m_arrSampler[0].GetAddressOf());
    }
    ```
    
    - Address mode
        - U축의 값을 구할때 1보다 클경우 처리하는 방법을 지정한다.
        - WRAP의 경우 감싼다는 의미로 1.5가 들어오면 0.5를 반환한다.
        - MIRROR모드는 대칭의 의미로 1.3이 들어오면 0.7를 반환한다. 즉 대칭적인 이미지를 그릴수 있게된다.
    - 필터의 차이
        - 이방선 필터(ANISOTROPIC)
            - 텍스처를 샘플링을 할떄 그 위치에 있는 색상을 그대로 가져오는게 아닌 주변 색상들이 평균값을 가져온다. 그래서 작은 이미지를 큰 화면에 불러오면 뿌옇게 보인다.
            - 현대에 있는 게임을 제작할때 많이 사용한다.
        - MIP_MAP_POINTER
            - 텍스처에 있는 색상을 그대로 가져온다. 이미지가 선명해지는대신 각이 지는 느낌이든다.
            - 80~90년대의 게임느낌을 살리는데 많이 사용된다.