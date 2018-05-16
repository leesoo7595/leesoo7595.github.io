---
layout: post
title: "알고리즘(다익스트라)"
date: 2018-02-27  00:00:00
img:
tags: [알고리즘]
---
> Hello Coding 그림으로 개념을 이해하는 알고리즘 책 정리

---

### 너비 우선 탐색 vs 다익스트라 알고리즘
- `너비 우선 탐색`은 가장 짧은 길, 즉 최단 경로를 찾을 때 사용한다.
- `다익스트라 알고리즘`은 최단 시간 경로를 구할 수 있는 알고리즘이다.

### 다익스트라 알고리즘
1. 가장 시간이 적게 걸리는 정점을 찾는다.
2. 이 정점의 이웃 정점들의 가격을 조사한다. 가격이 더 싼 경로가 존재하는지 확인 하고, 존재한다면, 가격을 수정한다.
3. 그래프 상의 모든 정점에 대해 이러한 일을 반복한다.
4. 최종 경로를 계산한다.

### 용어 설명
- `가중 그래프`의 각 간선에는 *가중치(Weight)* 을 가지고 있다.- 너비우선 알고리즘
- `균일 그래프`는 가중치가 없는 균일한 그래프를 의미한다. - 다익스트라 알고리즘

### 간선의 가중치가 음수인 경우
- 음의 가중치가 있으면, 다익스트라 알고리즘 사용이 불가하다. 앞서 어떤 정점을 처리하면 그 정점에 도달하는 더 싼 경로는 존재하지 않아야 한다. 그러나 음의 가중치가 있는 경우엔 예외 상황이 발생하기도 한다.

#### 벨만-포드 알고리즘(Bellman-Ford algorithm)
>  가중 유향 그래프에서 최단 경로 문제를 푸는 알고리즘이다. 이때 변의 가중치는 음수일 수도 있다. 다익스트라 알고리즘은 벨먼-포드 알고리즘과 동일한 작업을 수행하고 실행속도도 더 빠르다. 하지만 다익스트라 알고리즘은 가중치가 음수인 경우는 처리할 수 없으므로, 이런 경우에는 벨먼-포드 알고리즘을 사용한다. *위키 백과*

- 음의 가중치가 있는 그래프의 경우, 항상 무한 사이클을 생각해야 한다.
- 결론적으로 말해서 그 이유는 이전 노드까지 계산해둔 최소거리 값이 최소라고 보장할 수 없기 때문이다. 다익스트라는 정점을 지날수록 가중치가 증가한다 라는 사실을 전제하고 쓰여진 알고리즘이다. 하지만 음의 가중치가 있다면 정점을 지날수록 가중치가 감소할 수도 있다는 의미가 되므로 앞선 전제가 무너진다. 그러므로 다익스트라 알고리즘에서는 음의 가중치를 계산할 수 없다.
[출처](http://makefortune2.tistory.com/26)

#### 구현

```py
# 전체 그래프
graph = {}
graph["start"]={}
graph["start"]["a"]=6
graph["start"]["b"]=2
graph["a"]={}
graph["a"]["fin"]=1
graph["b"]={}
graph["b"]["a"]=3
graph["b"]["fin"] = 5
graph["fin"]={}
print(graph)

# 각 정점의 가중치 저장
# 정점의 가중치를 모르는 경우 무한대로
infinity = float("inf")
costs = {}
costs["a"]=6
costs["b"]=2
costs["fin"] = infinity
print(costs)

# 부모를 나타내줌
parents = {}
parents["a"]="start"
parents["b"]="start"
parents["fin"]=None
processed = []
print(parents)

# 가장 짧은 거리를 찾는 함수
def fine_lowest_cost_node(costs):
    lowest_cost = float("inf")
    lowest_cost_node = None

    # 모든 정점을 확인
    for node in costs:
          cost = costs[node]

        # 처리하지 않은 정점 중, 거리가 짧은 것이 있다면, 새로운 최저 정점으로 설정
        if cost < lowest_cost and node not in processed:  
            lowest_cost = cost
            lowest_cost_node = node
            print(f'low-{lowest_cost_node}/{lowest_cost}')

    return lowest_cost_node    

# 처리하지 않은 가장 싼 정점 찾기
node = fine_lowest_cost_node(costs)
print(f'node={node}')
# 모든 정점을 처리하면 반복문 종류
while node is not None:
    cost = costs[node]
    neighbors = graph[node]

    # 모든 이웃에 대해 반복
    for n in neighbors.keys():
        new_cost = cost + neighbors[n]
        print(f'new_cost={new_cost}')

        # 만약 이 정점을 지나는 것이 가격이 더 싸면, 정점 가격 갱신, 부모를 이 정점으로 설정,
        if costs[n]>new_cost:
            costs[n]=new_cost
            parents[n]=node

    # 정점을 처리한 사실을 기록
    processed.append(node)  
    print(f'processed= {processed}')
    # 다음으로 처리할 정점 찾아 반복
    node = fine_lowest_cost_node(costs)

print(costs)

```
