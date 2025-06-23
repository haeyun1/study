## Graph 종류
- Digraph (Directed) Graph : 방향이 있는 Graph
	- in-degree: 자기 vertex에 들어오는 edge의 개수
	- out-degree: 자기 vertex에서 나가는 edge의 개수
	edge가 자기 자신과 연결될 수 있다
	- head: edge가 출발하는 곳
	- tail: edge가 도착하는 곳
	
- Undirected Graph : 방향이 없는 Graph
	edge가 자기 자신과 연결될 수 없다
- Weighted Graph
- Not Weighted Graph
- Symmetric Digraph : 어떤 정점 u에서 v로 가는 간선 (u,v)가 존재하면, 반드시 v에서 u로 가는 역방향 간선 (v,u)도 존재해야 한다.
- Complete Graph : 그래프 내의 모든 서로 다른 두 정점 쌍 사이에 정확히 하나의 간선이 존재한다. 즉, 모든 정점이 다른 모든 정점과 직접 연결되어 있다.
	보통 방향이 없는 그래프임
- Dense Graph : many edges
	- Undirected
		complete graph이면 edge의 개수가 n(n-1) / 2
	- Directed
		complete graph이면 edge의 개수가 n(n-1) -> 방향이 있기 때문
- Sparse Graph : fewer edges
	edge가 없을 수도 있다
- spanning graph : 모든 vertex가 연결되어 있는 subgraph
	- spanning tree : spanning graph이면서 tree
- Strongly connected digraph
	- node u가 v에 도달할 수 있으면 v도 u에 도달할 수 있어야 함
- Directed acyclic graph (DAG) : 방향성은 있는데 cycle이 없는 graph
	- acyclic : cycle이 없음
- Tree : 모든 vertex들이 connected 되어 있으면서 cycle이 없는 graph

## Graph의 표현
- Adjacency Matrix Representation
	- Undirected graph : 자기 자신 및 연결되어 있지 않으면 0, 연결되어 있으면 1
		저장 공간이 O(V^2)만큼 듦 -> 방향이 없으므로 절반은 낭비되는 공간이므로 줄일 수 있음, adjacency list (linked list)를 사용함
- Adjacency List Representation
	- O(V+E) 만큼의 저장 공간 필요

## BFS
## DFS
## Topological Sort
finish time이 큰 것부터 정렬

## Strong Connected Component 찾기
SCC를 찾고자 하는 Graph의 edge 방향을 반대로 돌린 다음 finish time이 제일 느린 것부터 DFS

## Critical Path 찾기
eft, est 사용

## Articulation Point 찾기

## Minimum Spanning Tree 찾기
### Kruskal's Algorithm
edge weight를 오름차순으로 정렬한 다음에 cycle이 생기지 않도록 greedy하게 선택
### Prim's Algorithm
하나의 tree에서 뻗어나가는 가장 weight이 작은 node를 greedy하게 선택

## Shortest Path
input: directed graph with weight function
### Single-Source Shortest Path

#### Bellman-Ford Algorithm
가장 제약 조건이 작은 알고리즘, 음수 edge 허용, 음수 weight cycle 비허용
#### Dijkstra's Algorithm

### All-Pairs Shortest-Paths

#### Floyd-Warshall Algorithm

