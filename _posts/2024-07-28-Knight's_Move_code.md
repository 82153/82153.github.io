---
layout: article
title: "[구름 Level] Knight's Move"
date: 2024-7-28 11:12 +0900
category: Coding_Test
tags: [Conding Test]
sidebar:
  nav: cote
---
[문제](https://level.goorm.io/exam/43063/knight-s-move/quiz/1)
```python
def bfs(graph, start, n, m): # bfs 함수 구현
    visited = []
    route = []
    queue = [start]
    visited.append(start)

    # 나이트가 이동할 수 있는 방향
    dx = [2, 2, -2, -2, 1, -1, 1, -1] 
    dy = [1, -1, 1, -1, 2, 2, -2, -2]
 
    while queue: # 큐가 비어있지 않을 동안 반복
        tmp = queue.pop(0)
        route.append(tmp)
        for i in range(8):
            ny = tmp[0] + dy[i]
            nx = tmp[1] + dx[i]

            # 새로운 위치가 체스판 안에 있고, 아직 방문하지 않았을 경우
            if 0<=ny and n > ny and nx <m and nx >=0 and [ny, nx] not in visited:
                visited.append([ny, nx])
                queue.append([ny, nx])

                # 체스판에서 이전 위치에 1을 더한 값을 새로운 위치에 저장(즉, 이동한 횟수를 나타내게 됨)
                graph[ny][nx] = graph[tmp[0]][tmp[1]] + 1
    return route

# 체스판의 크기를 입력받음
n, m = map(int, input().split())

chess = [[0]*m for _ in range(n)]

# 나이트가 이동할 수 있는 위치들을 move에 저장
move = bfs(chess, [0,0], n, m)
cnt = 0

# 체스판에서 가장 큰수를 찾음(즉, 최대 이동횟수를 찾음)
for i in chess:
    cnt = max(cnt, max(i))

# 만약 체스판의 모든 칸으로 이동 가능할 경우
if len(move) == n*m:
    print('T{}'.format(cnt))

# 모두 이동할 수 없는 경우
else:
    print('F{}'.format(cnt))
```
