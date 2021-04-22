---
layout: post
title: '[BOJ] 세그먼트 트리(Segment tree)'
subtitle: 'segment tree, BOJ 2042'
categories: algorithm
tags: boj
usemath: true
comments: true
---


## 세그먼트 트리 (Segment tree)
태그: segment tree

## 목차
[Solution 1. 선형 탐색](#solution-1-선형-탐색)

[Solution 2. 세그먼트 트리](#solution-2-세그먼트-트리)

[1. 생성하기 (Initialization)](#1-생성하기-initialization)

[2. 검색하기 (Search)](#2-검색하기-search)

[3. 수정하기 (Update)](#3-수정하기-update)

[예제) BOJ 2042](#예제-boj-2042)

---

 세그먼트 트리(Segment tree)는 어떤 수열들의 특정 구간에 대한 부분합, 최소값, 최대값 등을 쉽게 구하기 위하여 사용하는 자료구조입니다.

 보통 부분합, 최소값 등을 구하고자 하는 질의(query)가 여러번 있을 때 사용합니다.

예를 들어, 배열(int arr[12])에 다음과 같은 값들이 있다고 가정하고 여기서 부분합(1~8)을 구해봅시다. 

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled.png)

```cpp
int arr[12] = {10, 7, 9, 0, 11, 7, 6, 5, 2, 13, 8, 6, 9};
```

---

### Solution 1. 선형 탐색

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%201.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%201.png)

 구간 1~8의 부분합을 구하고자 할 때, 기본적인 방법으로는 선형 탐색이 가능합니다. 우리가 기존에 알던 for문을 사용하여 부분합을 구할 수 있습니다.

```cpp
int left = 1, right = 8;
int ans = 0;
for(int i=left ; i<right ; i++){
	ans += arr[i];
}
```

 위의 코드를 통해 ans에는 7+9+0+11+7+6+5+2= 47가 들어가 있을 것입니다.

 이 해결책은 필요한 연산 구간만큼의 시간복잡도를 필요로 한다. $ O(right - left) $즉, 처음부터 끝까지 합을 구할 때는 $O(N)$의 시간복잡도를 가지게 됩니다.

 <span style="color:red"> 만약 부분합을 구할 일이 1회라면 합리적이지만, $k$번 구해야 한다면 $O(kN)$의 시간복잡도가 소요됩니다. </span>

 

---

### Solution 2. 세그먼트 트리

 부분합을 구할 일이 여러번 있을 때, Solution 1의 방법으로는 좋은 성능을 이끌어내기 어렵습니다. 따라서 중간에 특정 구간의 부분합을 트리형태로 관리하여 시간복잡도를 개선한 것이 세그먼트 트리입니다.

 세그먼트 트리의 구조는 다음 그림과 같이 나타낼 수 있습니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%202.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%202.png)

위 그림에서 파란색으로 색칠된 node는 leaf 노드이며 각자 원래 배열의 값을 갖고 있습니다. 그리고 그 위부터는 배열의 부분합을 갖게 됩니다.

 예를들어, 0~1은 index 0번~1번 원소의 합을, 4~6은 index 4번부터 6번까지 원소의 합을, 7~12는 index 7번부터 12번까지 원소의 합을 가지고 있을 것입니다. 

<span style="color:blue"> 만약 부분합이 아니라 최대값과 같은 다른 query를 원한다면, a~b에서 a번부터 b번까지의 최대값을 저장해놓으면 됩니다. </span>

 기존의 예제를 그대로 사용하여 부분합-세그먼트 트리를 구성하면 다음과 같습니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%203.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%203.png)

<span style="color:blue"> 파란색 node들을 살펴보면, 모든 leaf 노드들의 value는 array value와 동일한 것을 알 수 있습니다. </span>

 여기서 우리가 아까 찾고자 했던 부분합은 1~8이고 이는 다음 그림에서 색칠된 원소들을 합하면 됩니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%204.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%204.png)

 7+9+24+7 = 47

그러면 이제 어떻게 만들지, 어떻게 계산할지 문제와 코드를 통해 알아보죠!

---

<span style="color:red">※ 이 글에서는 편의를 위해 모든 tree node들을 1차원 배열에서 관리할 것이다!  따라서, root node의 index는 1입니다. </span>

<span style="color:red">left child는 현재 index * 2, </span> 

<span style="color:red">right child는 현재 index * 2 +1,</span>

<span style="color:red">parent는 현재 index / 2를 통해 계산할 수 있습니다.</span>

### 1) 생성하기 (Initialization)

  생성은 DFS와 동일합니다. 재귀적으로 child node들을 호출하다가, child node가 leaf node라면 값을 대입한 뒤에 반환해주면 됩니다.

```cpp
class Tree{
		int node[200000];
		
		int Init(int cur, int* arr, int left, int right){
	    if(left==right) node[cur] = arr[left];
	    int mid = (left+right)/2;
      return node[cur] = Init(cur*2, left, mid) + Init(cur*2+1, mid+1, right);
    }
};
```

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled.png)

 Tree의 노드는 몇개가 필요할까? 이는 full binary tree라고 가정하고 계산하는 것이 편할 것입니다. 

위 예시처럼 12개의 원소가 있는 배열 예제에서는 상단에 최대 15개의 부분합 node가 필요합니다. 즉, $2^{\lceil{log_2(N)}\rceil}$만큼의 node가 먼저 필요합니다. 여기에 leaves가 추가되어야 하므로 최종적으로는 다음과 같은 길이의 배열이 필요합니다.

> $2^{\lceil{log_2(N)}\rceil+1}$

 

생성하는 과정을 그림으로 나타내면 다음과 같이 그려볼 수 있습니다. 

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%205.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%205.png)

 원 위에 좌측 상단은 1차원 배열 내에서의 index, 그리고 left ~ right는 각각 함수 인자 left right를 의미합니다.

가장 먼저 처음으로 불렸을 때는 다음과 같을 것입니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%206.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%206.png)

 이 상황에서 left와 right는 같지 않으므로  Init(cur * 2, left, mid) 을 먼저 호출하겠죠.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%207.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%207.png)

역시 left와 right가 같지 않으므로 계속 left child를 재귀호출하여 다음과 같은 상황까지 도달하게 됩니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%208.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%208.png)

이제 index 16에 접근했을 때, left와 right는 모두 0으로 같아졌습니다! 이 경우, node[16] = arr[0]으로 값을 넣어줍니다. (node[cur] = arr[left])

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%209.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%209.png)

 이제 다시 node 8으로 돌아와서  Init(cur * 2, left, mid)의 함수가 종료되었으므로  Init(cur * 2+1, mid+1, right)을 호출하게 될거에요.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2010.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2010.png)

 여기서 left와 right가 같으므로 앞선 과정처럼 node[17] = arr[1] 을 넣어주게 됩니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2011.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2011.png)

 이제 node[8]에서 left child에 대한 init, right child에 대한 init이 모두 끝났으므로 다음 식을 계산할 수 있습니다.

```cpp
return node[cur] = Init(cur * 2, left, mid) + Init(cur * 2+1, mid+1, right);
//return node[cur] = 10 + 7 
```

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2012.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2012.png)

 이 과정에서 depth가 다른 leaf node가 생길 수 있습니다. 위 예제를 반복하다보면 아래 그림과 같이 홀수개의 정보를 포함한 node가 생길 수 밖에 없습니다. 

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2013.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2013.png)

  여기서 6~6은 left와 right가 동일하므로 child를 생성하지 않고 값을 저장합니다. (node[11] = arr[6])

 이 재귀 과정을 반복하면 최종적으로 다음과 같이 될 것입니다!

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2014.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2014.png)

 이제 Segment tree의 준비는 끝났습니다.

준비 과정은 tree의 node 수 만큼 순회를 요구하므로 $O(N)$ 의 시간복잡도를 갖습니다.

---

### 2) 검색하기 (Search)

검색은 필요한 부분만 재귀적으로 호출하면서 합을 구해나갈 수 있습니다. 첫 예제를 다시 살펴보죠.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2015.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2015.png)

우리가 구하고자 하는 1~8 구간의 부분합은 위에서 빨간색으로 색칠된 숫자들의 합을 구하면 됩니다. 

<span style="color:red">검색은 DFS처럼 재귀적으로 호출하되, 중간과정의 node가 계산에 필요한 node들의 합을 가지고 있다면 중단할 수 있습니다.</span>

```cpp
//cur : current node#
//left, right : 검색하고자 하는 구간
//start, end : node[cur]가 가지고 있는 정보의 구간
int Search(int cur, int left, int right, int start, int end){
	if(left<=start && right>=end) return node[cur];
	int mid = (start+end)/2;
	int sum = 0;
	if(left<=mid) sum += Search(cur*2, left, right, start, mid);
	if(right>mid) sum += Search(cur*2+1, left, right, mid+1, end);
	return sum;
}
```

left~right,  start~end의 의미를 잘 이해하면 쉽게 파악할 수 있습니다.

> 1차원 좌표에서 빨간색 막대는 해당 node에서 제외 시켜야 할 정보를 의미합니다.

1~8의 부분합을 검색하기 위해 가장 먼저, root node인 1번부터 살펴보도록 하죠.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2016.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2016.png)

 1번 노드를 보면 해당 node가 가지고 있는 103이란 값은 0(start)~12(end) 의 부분합입니다. 우리가 검색하고자 하는 영역 1(left)~8(right)외에도 필요없는 정보를 갖고 있으므로 자세히 쪼개서 더 살펴보아야 합니다.  

 특히, mid값 6을 기준으로 좌 우 모두 살펴봐야 하므로 left child, right child 모두 살펴봐야 해요.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2017.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2017.png)

 2번 node 부터 살펴보죠.  이 node가 갖고 있는 정보는 0(Start) ~ 6(End) 의 부분합입니다. 그런데 우리가 필요한 정보는 1(Left)~8(Right)까지 이므로 0~1은 필요가 없습니다. 결국 더 나눠야 합니다. 우리가 필요한 정보는 left child, right child 모두 갖고 있으므로 둘 다 살펴봐야 합니다.

 3번 node를 살펴보죠. 7~8의 부분합만 필요한데, 7~12의 정보를 갖고 있습니다. 마찬가지로 더 쪼개야 한다.

<span stype="color:blue"> 그런데 right child는 9~12라서 우리가 구하고자 하는 구간이 아닙니다. 그러므로 right child는 검사할 필요가 없습니다.</span>

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2018.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2018.png)

4번 노드는 여전히 mid를 기준으로 양쪽에 필요한 정보가 있으므로 쪼개야 한다.

5번 노드는 불필요한 정보가 없다! 5번 노드의 정보는 4(start)~6(end)의 부분합인데, 우리가 찾고자 하는 1(left)~8(right)에 속하므로 더 이상 child를 탐색할 필요가 없다.

6번 노드는 여전히 불필요한 정보가 존재하여 더 나눠야 합니다. 마찬가지로 right child는 살펴볼 필요가 없습니다.

다음 단계는 아래 그림과 같습니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2019.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2019.png)

 마지막으로 node 8번에서 right child만 선택하면 마무리됩니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2020.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2020.png)

---

### 3) 수정하기 (Update)

 수정하는 과정은 Init과 매우 유사합니다! 다만, update해야 하는 query는 구간이 아니고 1개일 경우만 해당합니다. (구간을 동시에 업데이트 하는것은 lazy propagation segment tree)

 Init은 left child, right child 둘 다 호출했다면, 1개의 원소만 update해야 하는 상황에서는 둘 중 하나만 부르면 됩니다. 바꿀 index가 left child인지, right child인지만 파악한 뒤 호출하면 됩니다.

```cpp
// cur : 현재 node
// index : 수정하고자 하는 위치
// val : 바꿀 값
// left, right : node[cur]가 담당하는 부분합 구간
void Update(int cur, int index, int val, int left, int right){
  if(left==right){
    node[cur] = val;
    return;
	}
	int mid = (left+right)/2;
	if(index <= mid) Update(cur*2, index, val, left, mid);
	if(index > mid) Update(cur*2+1, index, val, mid+1, right);
	node[cur] = node[cur*2] + node[cur*2+1];
}
```

만약 위 예제에서 4번 index의 값을 60으로 바꾸고 싶다고 예를 들어볼까요?

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2021.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2021.png)

 Root 노드부터 시작해서 바꾸고자 하는 index leaf node를 따라 들어가면 아래의 경로와 같을 것입니다. Index를 찾는 원리는 Search 할 때, 구간이 1개일 때와 같은 원리입니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2022.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2022.png)

이제 leaf node를 찾았다면 해당 값을 update 해줍니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2023.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2023.png)

 재귀 함수에서 마지막 단계의 (Leaf node) 함수를 종료한 뒤, parent node에서 갱신된 node와 기존의 node를 더해서 자신의 node를 갱신해주면 됩니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2024.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2024.png)

 이러한 재귀함수를 끝까지 반복하면 결과는 다음과 같을 것입니다.

![/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2025.png](/assets/images/posts/2021-02-27-algorithm-boj-segmenttree-1/Untitled%2025.png)

완성!

---

### 예제 BOJ 2042

> [BOJ2042](https://www.acmicpc.net/problem/2042) : 구간 합 구하기

이 문제는 그 동안 분석했던 생성하기(Init), 검색하기(Search), 수정하기(Update)를 하나의 class에 묶어서 사용하면 됩니다!

 문제에서는 1,000,000개의 원소들을 관리해야 하므로, $2^{\lceil{log_2(1,000,000)}\rceil+1} = 2^{21} = 2,097,152$ 만큼 필요합니다.

 계산이 귀찮다면 x4배만 넣어도 대부분 통과합니다!

```cpp
#include<stdio.h>
typedef long long int LLI;

class Tree{
        public:
        LLI node[2097152];

        LLI Init(int cur, LLI* arr, int left, int right){
                if(left==right){
                        node[cur] = arr[left];
                        return node[cur];
                }
                int mid = (left+right)/2;
                return node[cur] = Init(cur*2, arr, left, mid) + Init(cur*2+1,arr, mid+1, right);
        }
        LLI Search(int cur, int left, int right, int start, int end){
                if(left<=start && right>=end) return node[cur];
                int mid = (start+end)/2;
                LLI sum = 0;
                if(left<=mid) sum += Search(cur*2, left, right, start, mid);
                if(right>mid) sum += Search(cur*2+1, left, right, mid+1, end);
                return sum;
        }
        void Update(int cur, int index, LLI val, int left, int right){
                if(left==right){
                        node[cur] = val;
                        return;
                }
                int mid = (left+right)/2;
                if(index <= mid) Update(cur*2, index, val, left, mid);
                if(index > mid) Update(cur*2+1, index, val, mid+1, right);
                node[cur] = node[cur*2] + node[cur*2+1];
        }
};

int N,M,K;
LLI arr[1000000];
int main(){
        scanf("%d %d %d",&N,&M,&K);
        Tree* t = new Tree();
        for(int i=0 ; i<N ; i++){
                scanf("%lld",&arr[i]);
        }
        t->Init(1, arr, 0, N-1);

        int query;
        LLI a, b;
        for(int i=0 ; i<M+K ; i++){
                scanf("%d %lld %lld",&query, &a,&b);
                a--;
                if(query==1){
                        t->Update(1, a, b, 0, N-1);
                }else if(query==2){
                        b--;
                        printf("%lld\n",t->Search(1, a, b, 0, N-1));
                }
        }
        return 0;
}
```

<span style="color:red">※문제를 풀면서 주의할점</span>

- 수의 범위가 $-2^{63}$ ~ $2^{63}$이므로 integer로 해결할 수 없다.
- 문제에서는 index가 1번부터 시작하므로, 기존의 코드와 호환을 위해서는 1을 빼주어 0번부터 시작하도록 작성해야 한다.
