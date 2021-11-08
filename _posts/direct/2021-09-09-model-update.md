---
layout: post
title:  "[C++][Direct2D] Create Game Engine -4 update model"
summary: "Direct2D game engine"
author: moon
date: '2021-09-09 12:35:23 +0900'
category: Direct2D
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, oop, Direct2D, ui, grapic,engine
permalink: /blog/cpp/Direct2D/engine/update-model
usemathjax: true
---
## 물체이동시키기

---

- 기존의 방법대로 하면 정점의 정보를 업데이트 시켜 물체를 이동시켜야한다.
- 물체를 이동시키려면 GPU에 있는 버텍스 버퍼 메모리에 접근을 해서 변경해야한다.
- 직접적으로 버퍼를 변경하는 작업은 매우 느리며 치명적인 문제가 있다.
    - 시간이 지날수록 정점의 정보가 점점 더 커지게 된다.
    - 위치 좌표만 바꾸기 위해서 위치 정보만 전달할 수 없다.
    - 모든 정보를 다 준비해서 다시 보내야하는 매우 비효율적인 작업을 하게된다.
    - Direct에서는 모델을 공유할수 있는 자원으로 사용할 수 있다
    - 버텍스버퍼의 좌표를 직접 변경할 경우 공유 받은 모든 객체들에게 영향을 받게되어버리는 문제가 된다.
    
- 효율적인 처리를 위해서는 입력된 운동량(x로 얼마, y로 얼마)을 전해 모든 좌표로 전달해주는게 효율적이다.
- 이렇게 하면 원본의 모양을 지키며, 각각의 객체에 다른 옴직임을 전달할 수 있다.
- 문제는 버텍스 셰이더는 정점데이터 하나만 초기에 받을 수 있기 때문에 초기에 같이 전달 할 수 없다

### Register

---

- GPU에는 상수(b), 텍스처(t), 샘플러(s)를 저장할 수 있는 전용레지스터가 지정되어있다.
- 이 레지스터는 16byte로 데이터를 나누기때문에 float4로 데이터 타입을 이용하는게 좋다.
- 파이프라인 시작전에 레지스터에 16byte의 데이터를 가지고 있겠다는 걸 알려주는 과정을 Resource Binding이라고 한다.
- 각각의 레지스터를 이용하려면, 상수버퍼, 텍스처버퍼, 샘플러 버퍼와 역할이 해당 이용하려는 레지스터와 동일해야만한다.
- Resource Binding을 통해 상수레지스터를 이용하게 되면, 셰이더에서 해당 값들을 이용할 수 있게되고, 정점의 위치 변경을 효율적으로 할수있게된다.

### 상수버퍼

---

- 생성
    
    ```cpp
    	desc.ByteWidth = sizeof(Vec4);
    	desc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
    	desc.Usage = D3D11_USAGE_DYNAMIC; //상수값을 계속 변경할 예정이므로
    	desc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
    
    	if (FAILED(DEVICE->CreateBuffer(&desc, nullptr, g_CB.GetAddressOf())))
    	{
    		assert(nullptr);
    	}
    ```
    
- 데이터 추가
    
    ```cpp
    	D3D11_MAPPED_SUBRESOURCE tSub = {};
    	CONTEXT->Map(g_CB.Get(), 0, D3D11_MAP_WRITE_DISCARD, 0, &tSub);
    	Vec4 moveDir = Vec4(0.3f, 0.0f, 0.0f, 1.0f);
    	memcpy(tSub.pData, &moveDir, sizeof(Vec4));
    
    	CONTEXT->Unmap(g_CB.Get(), 0);
    ```
    
    - Mapping의 개념
        - 맵핑을 진행하게 되면, 직접 GPU 메모리에 접근하여 작성하는게 아니라, System memory에 저장되어있는 공간을 GPU 메모리에 연결을 하게 되고, GPU는 그 공간만큼 메모리를 예약해놓는다. 이 과정을 맵핑이라고 한다.
        - 코드상에서 작업을 할땐 System memory 에서 진행하게된다.
        
- 사용
    
    ```cpp
    CDevice::GetInst()->ClearTarget();
    
    	UINT stride = sizeof(VTX);
    	UINT offset = 0;
    	CONTEXT->IASetVertexBuffers(0, 1, g_VB.GetAddressOf(), &stride, &offset);
    	CONTEXT->IASetIndexBuffer(g_IB.Get(), DXGI_FORMAT_R32_UINT, 0);
    	CONTEXT->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
    	CONTEXT->IASetInputLayout(g_inputLayout.Get());
    
    	CONTEXT->VSSetConstantBuffers(0, 1, g_CB.GetAddressOf());//b0레지스터에 상수버퍼 바인딩
    
    	CONTEXT->VSSetShader(g_vsShader.Get(), nullptr, 0);
    	CONTEXT->PSSetShader(g_psShader.Get(), nullptr, 0);
    
    	//CONTEXT->Draw(3, 0);
    	CONTEXT->DrawIndexed(6, 0, 0);
    	//원은 어떻게 그릴까?
    	CDevice::GetInst()->Present();
    ```
    
    - 위와 같이 렌더함수를 작성하면 사각형이 전체적으로 오른쪽으로 이동해서 나오게된다.
    - 여기서 봐야할건 상수버퍼를 어떤 셰이더에서 사용할지를 지정하고 있다는것이다.
    - 즉, 모든 셰이더에서 상수버퍼를 이용하고 싶다면, 셰이더 마다 상수버퍼를 이용할 수 있도록 설정 해줄 필요가 된다.