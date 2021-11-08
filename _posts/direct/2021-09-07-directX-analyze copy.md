---
layout: post
title:  "[C++][Direct2D] Create Game Engine -3 DirectX Analysis"
summary: "Direct2D game engine"
author: moon
date: '2021-09-07 12:35:23 +0900'
category: Direct2D
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, Direct2D, ui, grapic,engine
permalink: /blog/cpp/Direct2D/engine/directX-analysis
usemathjax: true
---
# Direct 객체 분석

---

### ID3D11Device

---

- Direct는 GPU 를 컨트롤하기 위한 인터페이스이다.
- 다양한 기능을 사용하게 될경우 GPU에 명령을 내려야하는데 이걸 각기 다른 객체에서 하는것보다 이를 총괄하는 객체하나가 관리하는게 훨씬 효율적이다.
- 그 역할을 하는게 Device이다. 주로 GPU에서의 메모리할당을 담당하고 있다.

### ID3D11DeviceContext

---

- 원래는 Device에서 메모리 할당과 렌더링의 기능을 모두 담당했으나, 이 기능의 폭이 너무 커져 Device가 너무 거대해져 나눠서 탄생하게 되었다.
- DeviceContext는 주로 렌더링을 담당하는 함수들을 가지고 있다.
- 대부분 작업이 Device로 메모리 할당을하고 DeviceContext로 렌더링을 진행한다.

### D3D11Device, Context 생성

---

```cpp
hr = D3D11CreateDevice(nullptr, D3D_DRIVER_TYPE_HARDWARE, 
		nullptr, iFlag, 0, 0, 
		D3D11_SDK_VERSION, 
		&m_pDeivce, &eLevel, &m_pContext);
```

- 이중포인터로 받아가는 이유는 함수를 통해 생성된 객체의 주소를 받아와야하기 때문이다.
- 객체의 생성을 자신이 만든 프로그램에서 직접하는게 아닌 라이브러리에서 생성해주고 넘겨주기 때문이다.

### D3D11객체 소멸

---

```cpp
if (nullptr != m_pDeivce)
	{
		m_pDeivce->Release();
	}
	if (nullptr != m_pContext)
	{
		m_pContext->Release();
	}
```

- Delete가 아닌 Release함수를 따로 이용하는 이유
    - 같은 곳에서 생성되는게 아닌 한쪽 에서 생성된 객체를 포인터에 저장해서 이중포인터를 갖게 되는데, 자신이 만든 객체의 포인터와 DLL에서 생성된 객체 포인터의 페이지가 서로 달라. delete를 시킬경우 문제가 발생할 수도 있다.
- Reference Count
    - 다른 객체에서 현재 객체의 포인터를 얼마나 참조하고 있는지에 대해 체크한다. 참조하는 객체의 갯수에 따라 수가 변동되며, 이 수가 0일때 객체가 소멸된다.

### 스왑체인

---

- 역할
    - 게임내의 그림이 그려질때 블링크(화면의 감빡임)의 현상이 생기는걸 방지해주는 역할을 수행한다.
- 원리
    - 스왑체인 객체는 시스템 메모리에 생성이 된다.(프로그래머가 생성된 객체에서 직접연결이 아닌 DLL에서 생성된 객체를 통해 연결이된다.)
    - WINAPI와 달리 렌더링 데이터를 저장하는 버퍼가 GPU쪽에 생성이된다.(기본적으로 두개가 만들어진다. 근데 추가도 가능하다)
    - 이전과 달리 GPU메모리에 우리의 렌더링한 결과가 그려지기 때문에 시스템에 존재하는 window bitmap에 복사를 해주어야한다.
        - 왜 스왑체인이 2개를 가지고 있나요? 시스템 메모리 버퍼랑 이용해서 더블 버퍼링이 가능하지 않나요?
            - 데이터 전송을 프레임을 기준으로 동기화를 시켜 전송하게 되는데 FRONT가 완성이 되었을때만 전송을 할 수 있는게 아니기때문에 그래픽카드에서 두개의 버퍼를 이용해야한다.
    - 그래픽카드에서는 2개의 버퍼에 번갈아가며 그림을 그려 윈도우에서 완전한 그림을 그려지는 모습만 보여지도록 만든다.
- 생성 순서
    - 스왑체인 옵션값 구조체 입력
        - 무조건 윈도우 크기와 같을 필요는 없다.
        - 다만 그려질때 그림이 늘어지는 현상이 나올 수도 있다.
        
        ```cpp
        //스왑체인 옵션값 입력
        	DXGI_SWAP_CHAIN_DESC desc = {};
        	//버퍼 정보
        	desc.BufferDesc.Width = m_vResolution.x;
        	desc.BufferDesc.Height = m_vResolution.y;
        	//프론트버퍼 출력 비용
        	//만약 여러 프레임의 지원을 하고 싶으면
        	//스왑체인을 부수고 다시만드는 기능도 지원해야합니다.
        	desc.BufferDesc.RefreshRate.Numerator = 60;
        	desc.BufferDesc.RefreshRate.Denominator = 1;
        
        	//버퍼의 픽셀 포맷
        	desc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
        
        	//scanlining 옵션
        	desc.BufferDesc.ScanlineOrdering = DXGI_MODE_SCANLINE_ORDER::DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED;
        	desc.BufferDesc.Scaling = DXGI_MODE_SCALING::DXGI_MODE_SCALING_STRETCHED;
        
        	//버퍼의 용도
        	desc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
        	
        	//버퍼의 카운트(그냥 1을 넣자)
        	desc.BufferCount = 1;
        	
        	//출력 윈도우
        	desc.OutputWindow = m_hwnd;
        
        	//멀티 샘플링 수준
        	desc.SampleDesc.Count = 1;
        	desc.SampleDesc.Quality = 0;
        
        	//창인지 전체인지
        	desc.Windowed = true;
        
        	//스왑할때 어떻게 할지
        	desc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
        ```
        
    - 스왑체인 생성시킬 객체 얻기
        
        ```cpp
        //이과정은 그냥 설계가 이따구라 따라야한다.
        	IDXGIDevice* pIDXGIDevice = nullptr;
        	IDXGIAdapter* pAdapter = nullptr;
        	IDXGIFactory* pFactory = nullptr;
        
        	m_pDeivce->QueryInterface(__uuidof(IDXGIDevice), (void**)&pIDXGIDevice);
        	pIDXGIDevice->GetParent(__uuidof(IDXGIAdapter), (void**)&pAdapter);
        	pAdapter->GetParent(__uuidof(IDXGIFactory), (void**)&pFactory);
        
        	//스왑체인 생성
        	if (FAILED(pFactory->CreateSwapChain(m_pDeivce, &desc, &m_pSwapChain)))
        	{
        
        		return E_FAIL;
        	};
        //잊지 말고 해제해주자
        	pIDXGIDevice->Release();
        	pAdapter->Release();
        	pFactory->Release();
        ```
        

### View

---

- 뷰의 역할
    - 장치가 원하는 Data를 직접적으로 갑자기 전달하면(Data를 확인하지 못한 상태) 장치가 원하는 데이터인지 확신 할 수 없다.
    - View를 통해 해당 데이터가 장치가 원하는 데이터라는걸 증명해주는 역할을 한다.
    - View를 통해 확신할 수 있는이유
        - 애초에 View가 생성될 수 없는 타입일 경우 View의 생성자체가 실패하게된다.
        - 따라서 이 View가 생성되었다는 의미가 원하는 Data라는 증명이다.
    - 기본적인 렌더링을 위해서는 RTV, DSV를 가지고 있어야한다.
    - 또한 두 텍스처의 크기는 동일해야만한다.
- 뷰의 종류
    - Render Target View(RTV)
        - 스왑체인이 가지고 있는 텍스처중 어느것을 렌더링하질 결정하여 알려주는 역할을 함
    - Depth Stencil View(DSV)
        - 렌더링될 오브젝트의 깊이정보를 가지고 있는 텍스처임을 알려주는 용도로 사용되는 View
        - Depth Stencil
            - 깊이 검사라고도 하며, 물체가 겹쳐져 보이면 안되는부분이나, 굴곡에 따라 표시가 되어야할 부분과 되면 안될부분을 검사하는 방법
    - Shader Resource View(SRV)
    - Unordered Access View (UAV)
- 생성코드
    
    ```cpp
    int CDevice::CreateView()
    {
    	//RTV
    	ID3D11Texture2D* pBackBuffer = nullptr;
    	m_pSwapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (void**)&pBackBuffer);
    	if (FAILED(m_pDeivce->CreateRenderTargetView(pBackBuffer, nullptr, &m_pRTV)))
    	{
    		return E_FAIL;
    	}
    	pBackBuffer->Release();
    
    	//DRV
    	//DepthStencil 용 texture 생성
    	D3D11_TEXTURE2D_DESC desc = {};
    
    	desc.Width = (UINT)m_vResolution.x;
    	desc.Height = (UINT)m_vResolution.y;
    
    	desc.MipLevels = 1;
    	desc.ArraySize = 1;
    	//3바이트 깊이 1바이트 스텐실 따라서 이 포맷을 사용
    	desc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
    	desc.SampleDesc.Count = 1;
    	desc.SampleDesc.Quality = 0;
    
    	desc.Usage = D3D11_USAGE_DEFAULT;
    	desc.BindFlags = D3D11_BIND_DEPTH_STENCIL;
    
    	if (FAILED(m_pDeivce->CreateTexture2D(&desc, 0, &m_pDSTex)))
    	{
    		return E_FAIL;
    	}
    	if (FAILED(m_pDeivce->CreateDepthStencilView(m_pDSTex, nullptr, &m_pDSV)))
    	{
    		return E_FAIL;
    	}
    	
    	m_pContext->OMSetRenderTargets(1, &m_pRTV, m_pDSV);
    
    	return S_OK;
    }
    ```
    

### Viewport

---

- 역할
    - 게임내의 해상도와 윈도우 해상도가 일치않을 경우도 있다.
    - 이럴경우 어떻게 표현을 할지 정해야하는데 이걸 뷰포트가 수행한다.
- 주의할점
    - 뷰포트의 크기를 지정할때는 무조건 윈도우의 크기로 지정해야한다. 그래야만 게임내 해상도를 윈도우 크기에 맞추어 뷰포트가 늘렸다, 줄였다를 할수가 있다.
- 생성코드

```cpp
	D3D11_VIEWPORT VP = {};
	VP.TopLeftX = 0;
	VP.TopLeftY = 0;

	VP.Width = (UINT)m_vResolution.x;
	VP.Height = (UINT)m_vResolution.y;

	VP.MinDepth = 0;
	VP.MaxDepth = 1;

	m_pContext->RSSetViewports(1, &VP);
```