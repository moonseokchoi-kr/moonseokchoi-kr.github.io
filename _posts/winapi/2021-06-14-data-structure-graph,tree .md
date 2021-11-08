---
layout: post
title:  "[DataStructure] Graph, Tree"
summary: "graph and tree"
author: moon
date: '2021-06-14 12:35:23 +0900'
category: data_structure
thumbnail: /assets/img/posts/bitoper.PNG
keywords: cpp, stl, namespace
permalink: /blog/data-structure/graph
usemathjax: true
---
## Graph

---

- 데이터 가 담긴 하나의 자료를 Vertex라고 부른다.
- 이러한 vertex들의 연결관계를 나타낸 자료구조를 Graph라고 부른다.
- 그래프에서는 자료들의 연결을 통해 순회를 허용하고 있다.

## Tree

---

- Graph의 형태중 하나로 순회가 허용되지 않는 자료구조이다.
- 계층구조를 나타날때 유용하게 쓰이며, 운영체제의 디렉토리 구조를 나타낼때 사용된다.
- 부모-자식관계를 가지며, 한부모 밑에 얼마나 자식이 있느냐에 따라 트리를 부르는 명칭이 달라지기도 한다.
- 트리에서의 용어
    - Level, Depth : 트리의 깊이로 트리가 몇층까지 가지고 있는지 말한다.
    - Leaf node: 자식이 없는 노드를 의미.
    - Root node: 부모노드가 없는 노드,트리의 시작이자. 헤드 노드
- 트리의 종류
    - 이진트리
        - 한부모밑에 2개의 자식까지 가질 수 있는것
            - 이진탐색트리(Binary Search Tree)
                - 이진트리에서의 탐색을 중심으로 만든 트리
                - 데이터를 삽입할때부터 이진탐색이 가능하도록 만든다.
                - 중위순회를 하면 데이터가 정렬된 값을 얻을 수 있다.
                    - pre-order : 부모가 최우선 순위를 갖는다(다음 왼쪽 오른쪽 순)
                    - in - order : 왼쪽자식의 우선순위가 높다 다음 부모, 오른쪽 자식순이다.
                    - post-order: 자식의 우선순위가 높다(왼쪽1, 오른쪽 2, 부모 3)
                - 연산 수행시간
                    - 삽입 : $O(\log N)$
                    - 검색:  $O(\log N)$
                - 이진탐색트리의 문제점
                    - 데이터의 입력이 순차적으로 될때 한쪽으로 균형이 치우쳐져 의미가 없어짐
                    - 이를 위해 사용되는 자가균형 알고리즘들이 있다.(Red-Black, AVL)
                - 이진탐색에서의 중위순회
                    - 이진탐색에서 중위순회를 위해서는 선행자와, 후속자를 찾는 함수가 필요하다.
                        1. 왼쪽 자신이 있는지 체크 있다면 다운 없다면 그대로
                        2. 자신이 왼쪽노드라면 다음 후속자는 부모
                        3. 
                        - 자신이 왼쪽 노드라면 다음이 부모
                        - 만약 자신에게 오른쪽 자식이 있다면 2순위 없다면 자신 출력
                        - 그외라면 부모의 부모 출력
                        - 
            - 이진 탐색(Binary Search)
                - 정렬된 자료에서만 사용할 수 있다.
                - 중앙 인덱스를 기준으로 찾는 값이 크면 인덱스를 증가, 작으면 인덱스를 감소시키고 다시 중앙값을 구해 이과정을 반복한다.
                - 평균수행시간 : $O(\log n)$
                - Sudo-Code

                    ```cpp
                    Binary-Search(int start, int end, int value){
                    	if(start>end)
                    		return null;
                    	
                    	int m = start+end/2;

                    	if(Array[m]<value)
                    		return Binary-Search(m+1, end, value)

                    	if(Array[m]>value)
                    		return Binary-Search(start,m, value)
                    	else
                    		return Array[m]

                    }

                    ```

    - 완전이진트리
        - 한부모밑에 모든 자식이 2개인 트리
        - 배열로 구현이 용이하기 때문에 대부분 배열로 구현한다.
        - 왜냐하면 인덱스 연산으로 노드들의 연산을 찾을 수 있기 때문이다.
        - 주로 heap 자료구조를 구현하는데 많이 사용된다.

## Set

---

- 트리를 기반으로 만들어진 자료구조로 중복을 허용하지 않는 자료구조이다.
- 자가균형알고리즘이 적용되서 이진탐색시 엄청난 속도를 보여준다.
- 자료저장에 한계가 있기 때문에 생각보다는 많이 쓰이지 않는다.
- 주로 중복검사를 할때 많이쓰인다.

## Map

---

- Key와 Value라는 2개의 타입을 갖는 자료구조로, Key와 Value가 1:1 관계가 특징이다.
- 키를 통해 값을 찾는 구조로 원하는 값이 있을때 키를 통해 접근이 가능하다.
- 중복을 허용한다.
- C++에서는 Map을 라이브러리 제공
- 검색은 키값으로 검색하며 찾는 값이 없으면 end iterator를 반환한다.
- Map에서 자신이 만든 클래스나 구조체를 키값으로 사용하려면 비교연산을 오버로딩해야한다.
- 사용

    ```cpp
    #include<map>
    using namespace std::map, std::make_pair
    int main()
    {
    	map<const wchar_t*, int> mapData
    	//삽입
    	mapData.insert(make_pair(L"홍길동", 18));
    	mapData.insert(make_pair(L"이지혜", 20));
    	//검색(반복자를 리턴)
    	map<const wchar_t*, int>::iterator mapIter= mapData.find(L"홍길동");
    	
    	//키
    	mapIter->first
    	//값
    	mapIter->second
    }

    ```

    - 현재 예제는 문제가 있는데 바로 wchar_t*이다. 문자열을 위와 같이 비교해서 찾을 경우 문자열의 구성이 아닌 문자열의 시작주소를 비교하여 찾기 때문에 같은 단어라도 주소가 다르면 다르다고 컴파일러가 생각한다.
    - 이를 방지하기 위해 문자열 객체인 <string>을 이용하는게 좋다.
        - wstring : 2byte문자열
        - string : 1byte 문자열