---
layout: post
title: "알고리즘 - 프로그래머스 데모 테스트 문제1"
date: 2018-08-28 00:00:00
img:
categories:
- Algorithm/DataStructure
tags: [알고리즘/자료구조]
---

## 문제
```
문제 설명
길이가 n인 배열에 1부터 n까지 숫자가 중복 없이 한 번씩 들어 있는지를 확인하려고 합니다.
1부터 n까지 숫자가 중복 없이 한 번씩 들어 있는 경우 true를, 아닌 경우 false를 반환하도록 함수 solution을 완성해주세요.

제한사항
배열의 길이는 10만 이하입니다.
배열의 원소는 10만 이하의 자연수입니다.
입출력 예
arr	result
[4, 1, 3, 2]	true
[4, 1, 3]	false
입출력 예 설명
입출력 예 #1
입력이 [4, 1, 3, 2]가 주어진 경우, 배열의 길이가 4이므로 배열에는 1부터 4까지 숫자가 모두 들어 있어야 합니다. [4, 1, 3, 2]에는 1부터 4까지의 숫자가 모두 들어 있으므로 true를 반환하면 됩니다.
입출력 예 #2
[4, 1, 3]이 주어진 경우, 배열의 길이가 3이므로 배열에는 1부터 3까지 숫자가 모두 들어 있어야 합니다. [4, 1, 3]에는 2가 없고 4가 있으므로 false를 반환하면 됩니다.
```

## 코드
```python
# 정확성 통과, 효율성 통과
def solution(arr):
    result_dic = {value:False for value in range(1,len(arr)+1)}
    for value in arr:
        if value in result_dic:
            result_dic[value] = True

    answer = all(value for value in result_dic.values())
    return answer

# 정확성 통과, 효율성 1개 통과못함
def solution(arr):

  result_dic = {value:False for value in range(1,len(arr)+1)}
  answer = True

  for value in arr:
      if value > len(arr) or value not in result_dic:
          answer = False
          break
      if result_dic[value]:
          answer = False
          break
      else:
          result_dic[value] = True
  return answer

# 정확성 통과, 효율성 문제 있던 부분 통과 그러나 다른 효율성 통과 못함
  def solution(arr):

      result_dic = {}
      answer = True

      for value in range(1,len(arr)+1):
          if value > len(arr) or value not in arr:
              answer = False
              break
          if result_dic.get(value):
              answer = False
              break
          else:
              result_dic[value] = True
      return answer
      
```
