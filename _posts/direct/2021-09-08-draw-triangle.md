---
layout: post
title:  "[C++][Direct2D] Create Game Engine -3 Graphics Pipeline"
summary: "Direct2D game engine"
author: moon
date: '2021-09-08 12:35:23 +0900'
category: Direct2D
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, Direct2D, ui, grapic,engine
permalink: /blog/cpp/Direct2D/engine/graphics-pipline
usemathjax: true
---
# Direct에서 물체가 그려지는 과정(Graphics Pipeline)

---

- 물체를 그리는 과정은 파이프라인에서 시작해, 백버퍼, 프론트 윈도우로 데이터를 넘겨 최종적으로 그림이 보이게 된다.
    
    ### Graphic Pipeline
    
    ---
    
    - Input Assembler
        - 입력에 대한 준비를 하는 단계로, 정점 버퍼, 픽셀버퍼등의 메모리를 준비하고 연결하는 단계이다. 그리고 렌더링에 필요한 데이터를 전달한다.
    - Vertex Shader Stage
        - 어셈블러에서 정점을 처리 하 고 변환, 스키닝, 모핑 및 정점 별 조명 등의 정점 작업을 수행 합니다.
        - 정점 셰이더는 항상 단일 입력 정점에 대해 작업을 수행 하 고 단일 출력 정점을 생성 합니다.
    - Hull Shader Stage
        - 각 입력 패치 (쿼드, 삼각형 또는 선)에 해당 하는 기 하 도형 패치 (및 패치 상수)를 생성 하는 프로그래밍 가능한 셰이더 단계입니다.
    - Tesellation Stage
        - 기하 도형 패치를 나타내는 도메인의 샘플링 패턴을 만들고 이러한 예제를 연결 하는 작은 개체 (삼각형, 요소 또는 선) 집합을 생성 하는 고정 함수 파이프라인 단계입니다.
    - Domain Shader
        - 각 도메인 샘플에 해당 하는 꼭 짓 점 위치를 계산 하는 프로그래밍 가능한 셰이더 단계입니다.
    - Geometry Shader
        - 꼭짓점을 입력으로 사용하여 애플리케이션 지정 셰이더 코드를 실행하고 출력 시 꼭짓점을 생성하는 기능을 실행합니다.
        - 주로 파티클 기능을 만드는데 사용한다.
    - OutputStream
    - Rasterizer
        - 기본 형식은 픽셀로 변환 되 고 각 기본 형식에는 정점 별 값을 보간 합니다.
        - 래스터화에는 뷰에 클리핑 꼭지점이 포함 됩니다. 즉, 원근감을 제공 하기 위해 z로 나누기를 수행 하고, 기본 형식을 2D 뷰포트에 매핑하고, 픽셀 셰이더를 호출 하는 방법을 결정 합니다
    - pixel shader
        - 화면의 색상을 지정하는 셰이더 선택된 타겟의 해당하는 픽셀의 색상을 지정한다.
        - 여기서 지정된 색상을 통해 BlendState에서 보간을 진행하고 최종색상이 지정된다.
    - Output Merge State Stage
        - Depth Stencil State
            - 깊이 텍스쳐에 따라 물체의 깊이를 평가하는 단계
            - Direct에서는 z축의 값에 따라 깊이를 평가해 깊이가 얕은부분을 그리고 깊은 부분은 그리지 않는다.
            - 그렇기 때문에 물체가 곂쳐지게 되면 깊이가 앝은 물체가 보이게 된다.
            - 이러한 깊이버퍼덕분에 렌더링 순서에 영향을 받지 않는다.
        - Blend State
            - 색상을 조합하는 단계 보간을 통해 메쉬의 색상을 결정한다.
    
    ### 자세한 정보는 여기에서!
    
    [그래픽 파이프라인 - Win32 apps](https://docs.microsoft.com/ko-kr/windows/win32/direct3d11/overviews-direct3d-11-graphics-pipeline)
    
    - 코드로 표현
        
        ```cpp
        //이전 프레임의 데이터를 초기화
        	CONTEXT->ClearRenderTargetView();
        	CONTEXT->ClearDepthStencilView();
        
        	//InputAssembler
        	CONTEXT->IASetIndexBuffer();
        	CONTEXT->IASetVertexBuffers();
        
        	//Vertex
        	CONTEXT->VSSetShader();
        	//Tesellation
        	CONTEXT->HSSetShader();
        	CONTEXT->DSSetShader();
        
        	//Geometry
        	CONTEXT->GSSetShader();
        
        	//Resteraizer
        	CONTEXT->RSSetState();
        	
        	//OutputMergeState
        	CONTEXT->OMSetBlendState();
        	CONTEXT->OMSetDepthStencilState();
        
        	/// 
        	/// Draw 전까지만 순서에 관계없이 설정만 해주면 된다.
        	///
        	CONTEXT->Draw();
        	
        	//
        	//마지막에 스왑체인으로 백과 프론트를 바꿔주면 종료
        ```
        
    
    ### Shader Stage
    
    ---
    
    - 사용자가 만드는 함수들로 해당 스테이지에 어떤식으로 작동을 할지 직접 사용자가 작성해야만 하는 스테이지이다.
    - 대부분 스테이지 이름에 Shader가 들어간다.
    
    ### 고정 스테이지
    
    ---
    
    - 사용자의 개입이 불가능한 단계로 간단한 설정값을 넣을 수 있을 뿐 하드 코딩은 불가능하다.
    
    ### 삼각형 그리기
    
    ---
    
    ### 버퍼채우기
    
    ---
    
    - 삼각형을 그리기 위해서는 IA에 데이터를 입력해야한다.
    - 하지만 우리는 C++로 시스템에서 코딩을 하지 직접적으로 GPU메모리에 접근해서 작업을 하지 않는다.
    - 따라서 메모리에 만든 데이터를 GPU에 보내줘야하는데 이 데이터를 버퍼라고 부른다.
    - 정점정보가 저장되면 버텍스버퍼, 인덱스가 저장되면 인덱스 버퍼, 어떤 구조가 필요할때 사용하면 구조화 버퍼이다.
    - 이러한 버퍼를 생성하고 관리하기 위해 ID3D11Buffer 객체를 이용한다.
    - 버퍼를 생성하기 위해서는 버퍼의 설정값을 구조체에 채워 넘겨줘야한다(D3D11_BUFFER_DESC)
    - 버퍼에 값을 채워넣는건 처음 버퍼가 생성될때밖에 불가능하다(생성하고 나중에 초기값 설정이 불가능)
    - 버퍼 내용채우는 코드
    
    ```cpp
    //버퍼 설정
    	//DYNAMIC - WRITE = 내용을 변경할 수 있음
    	//STAGING - READ =  내용을 읽어서 가져올수 있음
    	D3D11_BUFFER_DESC desc = {};
    	desc.ByteWidth = sizeof(VTX) * 3;
    	desc.CPUAccessFlags = 0;
    	desc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
    	desc.Usage = D3D11_USAGE_DEFAULT;
    
    	//넣어줄 초기 데이터 매칭
    	D3D11_SUBRESOURCE_DATA sub = {};
    	sub.pSysMem = arrVTX;
    ```
    
    ### Index Buffer
    
    ---
    
    - 사각형이나 원같이 여러 삼각형을 이용해 도형을 만들게 되면, 중복되는 정점들이 생길 수 밖에 없다.
    - 이러한 중복되는 정점들을 모두 Vertex 버퍼에 저장하게되면, 엄청난 메모리를 사용하게 된다.
    - 그래서 Direct에서는 Index버퍼라는걸 이용해 메모리 절약을 하게된다.
    - Index버퍼를 이용하게 되면 Direct는 인덱스버퍼에서 정점의 크기만큼 잘라서 그림을 그리게 된다.
    - 인덱스버퍼는 정점의 인덱스를 저장하게 된다. 이 인덱스의 순서로 Topology에서 지정된 도형으로 그림을 그리게된다.
    - 예를 들어 사각형을 삼각형 2개로 그리게된다면 필요한 점은 4개지만, 중복적으로 점 2개 사용되 총 6개가 필요하다.
    - 그러므로 인덱스 버퍼가 작성된다면 0,1,2,0,2,3 이렇게 작성되고 이 순서대로 삼각형이 그려진다.
    
    ### 셰이더 작성
    
    ---
    
    - 입력데이터를 설정 했으면 이제 쉐이더 단계를 실행해야하는데 이 단계는 CPU가 아닌 GPU에서 실행되는 과정이다.
    - 이 과정에 필요한 함수들을 작성하는건 C++로 할 수 없다. 대신 HLSL(High Level Shading Language)를 통해 작성해야한다.
    - 이렇게 작성한 함수들을 쉐이더라고 한다.
    - HLSL은 C언어 스타일을 따르기 때문에 C언어로 작성해야한다.
    
     
    
    ### Semantic
    
    ---
    
    - HLSL에서 사용하는 지시어로 해당 변수가 어떠한 정보를 가지고 있는지 명시해주는데 사용한다.
    - 이걸 사용하는 이유는 셰이더 마다 필요한 정보가 다르기 때문이다. 만약 Semantic이 없다고 하면 나중을 위해 설정해놓은 구조가 추가 될때마다, 예전 셰이더를 갱신하는 매우 비효율적인 일이 진행된다.
    - 따라서 사용자가 정의한 C++파일에서 필요한 정보만 가져와 셰이더를 만들기위해 Semantic을 사용해야한다.
    
    ### InputLayOut
    
    ---
    
    - Semantic으로 위치와 색상등의 정보를 지정했으나 C++단계에서 우리는 뭐가 색상정보인지, 위치정보인지를 알려준적이 없다.
    - 이를 알려주기위해 InputLayout 구조체가 필요로 하다.
    - 서로다른 구조 입력구조가 있을때마다 각각에 맞게 하나씩 필요로하며, 이걸 작성해야만 정상적인 렌더링이 진행이 된다.
        
        ```cpp
        D3D11_INPUT_ELEMNET_DESC tLayout[2] = {};
        
        tLayout[0].SementicName = "POSISTION"//지정할 시멘틱 이름
        tLayout[0].SementicIndex = 0//동일한 시멘틱을 여러개 받아오고 싶을때 사용
        tLayout[0].AlignedByteOffset = 0;//해당 시멘틱의 오프셋
        tLayout[0].Format = DXGI_FORMAT_R32G32B32_FLOAT//픽셀포맷으로 시멘틱의 정보 사이즈를 알려주야함
        //인스턴스에 관련된 부분은 추후설명
        tLayout[0].InputSlot = 0;
        tLayout[0].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
        tLayout[0].InstanceDataStepRate = 0;
        
        tLayout[1].SementicName = "COLOR"//지정할 시멘틱 이름
        tLayout[1].SementicIndex = 0//동일한 시멘틱을 여러개 받아오고 싶을때 사용
        tLayout[1].AlignedByteOffset = 12;//해당 시멘틱의 오프셋
        tLayout[1].Format = DXGI_FORMAT_R32G32B32A32_FLOAT//픽셀포맷으로 시멘틱의 정보 사이즈를 알려주야함
        //인스턴스에 관련된 부분은 추후설명
        tLayout[1].InputSlot = 0;
        tLayout[1].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
        tLayout[1].InstanceDataStepRate = 0;
        
        //레이아웃이 필요한 셰이더를 지정
        DEVICE->CreateInputLayout(tLayout, 2, g_vsBlob->GetBufferPointer(), g_vsBlob->GetBufferSize(), g_LayOut.GetAddressOf())
        ```
        
    
    ### 정점셰이더(Vertex Shader)
    
    ---
    
    - input assembler에 입력된 정점에 대한 작업을 실행하는 셰이더이다.
    - 물체에 대한 크기, 이동, 회전등에 대한 연산을 실시한다.
    - 정점에 대한 로컬, 월드, 뷰, 투영변환을 실시한다.
    - 모든 정점에 대해 한번씩 실행하게된다.
    
    ### 픽셀셰이더(Pixel Shader)
    
    ---
    
    - 렌더링되는 영역에 해당하는 모든 픽셀에 적용되는 셰이더
    - 대부분 색과 관련된 작업을 수행한다.
    
    - 작성한 셰이더
    
    ```cpp
    struct VTX_IN
    {
    	float3 vPos : POSITION;//위치라 알려줌
    	float4 vColor : COLOR;//색이라 알려줌
    };
    
    struct VTX_OUT
    {
    	float4 vPosition : SV_Position;
    };
    
    VTX_OUT VS_Std(VTX_IN _in)
    {
    	VTX_OUT output = (VTX_OUT)0.f;
    	output.vPosition =float4( _in.vPos,1.f);
    
    	return output;
    }
    
    ///////////////////////////////////////////////////
    /// 
    /// Rasterizer
    /// 
    /// vertex shader 에서 전달한 SV_Position을 기반으로 위상구조, 삼각형에 맞는 
    /// 면적을 구하고 정점들을 연결해 삼각형으로 모델을 작성한다
    /// 
    /// 
    ////////////////////////////////////////////////// 
    
    float4 PS_Std(VTX_OUT _in):SV_Target
    {
    	float vOutColor = (float4) 0.f;
    	
    	return vOutColor;
    }
    ```
    
    - 셰이더 가져오기
        - 셰이더를 불러오기 위해서는 몇가지 설정이 필요하다.
            - Client가 셰이더의 위치를 가져와야한다(리소스처럼)
            - 바로 부를 수 있는게 아닌 D3D11Blob이라는 객체를 통해 부르게된다.
            - Blob객체는 문자열 데이터를 저장하는 버퍼정도 라고 생각하면 편하다.
        
        ```cpp
        D3DCompileFromFile(
        //쉐이더 경로,
        //nullptr,
        //설정된 매크로(D3D_COMPILE_STANDART_FILE_INCLUDE
        //"VS_Std" 셰이더의 진입점
        //"vs_5_0" 셰이더 문법 레벨
        //D3DCOMPILE_DEBUG 옵션값
        //옵션값 2
        //blob객체에 결과 저장
        //오류결과 저장
        )
        ```
        
- 셰이더 만들기
    - DEVICE객체를 통해 셰이더를 생성한다.
    - 무조건 compile→ create 로 진행된다.
    
    ```cpp
    DEVICE->CreateVertexShader(//버퍼의 주소, //크기, nullptr, //셰이더 객체)
    //픽셀셰이더도 똑같다.
    ```
    

### 렌더링 진행

---

- 렌더링을 진행하기 위해 Context에 설정을 해줘야할 값들이 존재한다.

```cpp
CONTEXT->IASetVertexBuffer(0,1, g_VB.GetAddressOf(), &stride, 0);
//여기서 주목해야할건 3,4,5인자
//3번은 버텍스버퍼의 주소
//4번은 버퍼하나의 사이즈
//5번은 버퍼의 시작위치이다.
CONTEXT->IASetInputLayOut(g_Layout.Get());
//위에서 설정한 Layout설정

//어떻게 정점들을 연결할건지에 대해 입력을 받음
CONTEXT->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST);
```

- 화면을 클리어 해주는 작업도 진행해야한다.

```cpp
Device->ClearRenderTargetView(m_pRTV.Get(),//지울 색상);
//깊이텍스쳐의 깊이를 다시설정하는 함수
//깊이는 0~1사이이다(가까운게 0, 제일먼게 1)
//0일경우 깊이 테스트를 통과할 수 없다.
Device->ClearDepthStencilView(m_pDSV.Get(), D3D11_CLEAR_DEPTH|D3D11_CLEAR_STENCIL, 1, 0 )
```

- 버텍스 버퍼를 설정했으므로 셰이더도 설정한다

```cpp
CONTEXT->VSSetShader(g_vs.Get(), nullptr,0);
CONTEXT->PSSetShader(g_ps.Get(), nullptr,0);
```

- rasterizer, Output Merge State는 기본값이 존재하기 때문에 별도로 설정하지 않는다면 기본설정으로 렌더링하게 된다. 여기서는 기본설정으로 진행하겠다.
- 이제 렌더링을 위한 준비가 완료 되었으니 렌더링을 진행한다.

```cpp
//인덱스 버퍼를 사용하지 않는 렌더링
CONTEXT->Draw(3,0);
//인덱스 버퍼를 사용하는 렌더링
CONTEXT->DrawIndexed(4,0);
//백퍼의 그려진 그림을 표시하는함수 
CDevice::GetInst()->Present();
```

### 삼각형 색상정보 이용하기

---

- 각 정점에는 색상이 지정되어 있는데 현재는 픽셀 셰이더에서 색상을 지정하여 해당색상으로 출력중이다.
- 정점에 지정된 색상을 이용해 삼각형을 칠해보자
    
    ```cpp
    #ifndef _STD2D
    #define _STD2D
    
    struct VTX_IN
    {
    	float3 vPos : POSITION;
    	float4 vColor : COLOR;
    };
    
    struct VTX_OUT
    {
        float4 vPosition : SV_Position;
        float4 vColor : COLOR;
    };
    
    VTX_OUT VS_Std(VTX_IN _in)
    {
    	VTX_OUT output = (VTX_OUT)0.f;
    	output.vPosition =float4( _in.vPos,1.f);
        output.vColor = _in.vColor;
    	return output;
    }
    
    float4 PS_Std(VTX_OUT _in):SV_Target
    {
        float4 vOutColor = _in.vColor;
    	
    	return vOutColor;
    }
    
    #endif
    ```
    
- 위와 같이 셰이더 코드를 수정하면아래와 같이 그려진다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ca725de9-0aa4-41b4-8cde-b0a8ff12978c/Untitled.png)
    

- 이렇게 그려지는 이유는 Rasterizer 단계와 Pixel Shader때문이다.
    - 정점에 저장된 색상정보를 기반으로 Rasterizer에서 보간을 진행하고 칠해질 픽셀들을 매핑한다.
    - 각 픽셀에 대해 픽셀셰이더가 실행되는데 입력으로 들어온 색상을 그대로 출력하는걸 선택했으므로 보간된 색상이 그대로 출력된다.