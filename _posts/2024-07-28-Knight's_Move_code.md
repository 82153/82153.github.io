```python
def bfs(graph, start, n, m):
    visited = []
    route = []
    queue = [start]
    visited.append(start)
    dx = [2, 2, -2, -2, 1, -1, 1, -1]
    dy = [1, -1, 1, -1, 2, 2, -2, -2]
 
    while queue:
        tmp = queue.pop(0)
        route.append(tmp)
        for i in range(8):
            ny = tmp[0] + dy[i]
            nx = tmp[1] + dx[i]
            if 0<=ny and n > ny and nx <m and nx >=0 and [ny, nx] not in visited:
                visited.append([ny, nx])
                queue.append([ny, nx])
                graph[ny][nx] = graph[tmp[0]][tmp[1]] + 1
    return route

n, m = map(int, input().split())

chess = [[0]*m for _ in range(n)]
move = bfs(chess, [0,0], n, m)
cnt = 0
for i in chess:
    cnt = max(cnt, max(i))
if len(move) == n*m:
    print('T{}'.format(cnt))
else:
    print('F{}'.format(cnt))
```
