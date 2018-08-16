---
layout: post
title: "탐색트리 - 이진탐색(Binary Search)"
date: 2018-08-14 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---
> 파이썬과 함께하는 자료구조의 이해 책 정리

----

## 이진탐색 (Binary Search)
- 1차원 리스트에 데이터가 정렬되어 있을 때, 주어진 데이터를 효율적으로 찾는 알고리즘
- 데이터가 정렬되어 있지 않다면, 주어진 데이터를 찾기 위해 리스트의 모든 원소들을 차례대로 검색하는 순차탐색을 수행해야 한다. 탐색 연산에 최악의 경우 **O(N)** 시간이 소요된다. **이진탐색은 데이터를 미리 정렬하여 최악경우에도 log<sub>N</sub> 항목 비교만 하는 매우 효율적인 탐색 방법**이다.


### 수행시간
- **T(N)** : N개의 원소가 정렬된 리스트에서 이진탐색을 하는데 수행되는 원소 비교 횟수
- **T(N) = T(N/2) + 1** : 1번의 비교 후에 리스트의 1/2, 즉 앞부분이나 뒷부분을 재귀호출
- **T(N) = T(N/2) + 1 + T(N/2<sup>2</sup>) + 2 + ..... k = T(N/2<sup>k</sup>) + k  = T(1) + k** , N = 2<sup>k</sup>라고 가정하면 **T(N) = 1 + log<sub>2</sub>N = O(logN)**
- 단점 : 삽입삭제가 빈번하면 정렬을 유지하기 위해 시간이 오래 걸린다. 1회의 삽입 삭제 연산 수행시 최악의 경우 O(N) 시간이 소요된다.

### 코드
```python
import random

def binary_search(left, right, t):
    if left > right:
        # 탐색실패 ( 즉 , t가 리스트에 없음)
        return None

    # 리스트에서 탐색할 부분의 중간 항목의 인덱스 계산
    mid = (left + right) // 2

    if a[mid] == t:
        # 탐색 성공
        return mid

    if a[mid] > t:
        # 앞부분 탐색
        binary_search(left, mid - 1, t)
    else:
        # 뒷부분 탐색
        binary_search(mid + 1, right, t)


if __name__ == '__main__':
    a = [random.randint(1, 100) for _ in range(20)]
    print(binary_search(0, 19, 66))

```
