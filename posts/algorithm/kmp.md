---
layout: post
title:  "KME"
date:   2019-01-02 02:00:00
author: Inhyuk
category: Algorithms
tags:	Algorithms
cover:  "/assets/instacode.png"
name: kmp.md
---

KMP
====

- **kmp 알고리즘**: 한 문자열이 다른 문자열에 속하는 지를 판단할 때 쓰이는 알고리즘
- 시간복잡도: O(N+M) / 공간복잡도: O(N+M). N: 찾고자하는 문자열의 길이, M: 서칭 대상 문자열


원리
-----

- 원리는 간단하다. 어떤 문자열이 있을 때, 그 문자열의 prefix와 suffix가 갖고 있는 정보를 활용하는 것이다.

- 두 문자열 A(abaaabaab)와 B(abaab)가 있고 A에서 B를 체크하고자 한다.
- index = 0에서 index=3인 abaa는 같다. 그러나 index=4에서 다르다(a != b)
- 이때 abaa의 prefix와 suffix가 같은 최대 값은 a이다. 그렇기 때문에 B(abaab)을 한 칸만 이동할 필요가 없다.
  - 만약 그렇게 해서 값을 찾는다면 abaa가 A[1:4]와 같아야 한다. 그 말은 abaa의 prefix=suffix인 경우는 최소한 aba를 포함하고 있어야 한다.
  - 같은 생각으로 하면 결국 prefix와 같은 suffix에 끼워 맞춰서 시작하면 된다.
- 이 방법을 찾을때까지 반복한다.

- 시간 복잡도는 O(M+N)이다


prefix 찾기
------

- 위의 B(abaab)에서 필요한 것은 어떤 B의 길이 이하의 x에 대하여 만들어진 부분 문자열 B[0:x]에서 prefix=suffix의 최대값을 찾아야 한다.
- 이 것도 위에 적힌 원리를 이용하면 된다.
- 방법은 아래의 코드를 참고.


- - -

예제
===

- [예제 링크](https://leetcode.com/problems/implement-strstr/submissions/)

```go
func strStr(haystack string, needle string) int {
    if len(needle) == 0 {
        return 0
    }

    p := make([]int, len(needle))
    compareIdx:= 0
    for i:=1; i<len(p); i++{
        if needle[i] == needle[compareIdx]{
            compareIdx++
            p[i] = compareIdx
        } else {
            if compareIdx == 0 {
                p[i]=0
            } else {
                compareIdx = p[compareIdx-1]
                i--
            }
        }
    }

    compareIdx = 0
    for i:=0; i<len(haystack); i++{
        if haystack[i] != needle[compareIdx]{
            if compareIdx != 0 {
                compareIdx = p[compareIdx-1]
                i--
            }
        } else {
            compareIdx++
            if compareIdx == len(needle){
                return i-len(needle)+1
            }
        }
    }
    return -1
}
```
