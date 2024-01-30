---
title: Dijkstra算法
tags: [algorithm, dijkstra]
sidebar_label: Dijkstra算法
sidebar_position: 2
---

# Dijkstra算法

----
**Algorithm: Dijkstra**

**Input**: Directed graph $G=(V,E,W)$ with weight.

**Output**: All the shortest paths from the source vertex s to every vertex $v_i \epsilon (V-\{s\})$

1: $S \leftarrow \{s\}$
2: $dist[s,s] \leftarrow 0$
3: for $v_i \epsilon \{s\}$ do
4: ____$dist[s,v_i] \leftarrow w(s, v_i)$ (when 5: $v_i$ not found, $dist[s, v_i] \leftarrow \infty $)
5: while $V-S\not ={\varnothing}$ do
6: ____find $min_{v_j\epsilon V} dist[s, v_j]$ from the set $V-S$
7: ____$S \leftarrow S \cup \{v_j\}$
8: ____for $v_i \epsilon V - S$ do
9: ________if $dist[s, v_j] + w_{j,i} < dist[s, v_i]$ then
10: ___________$dist[s, v_i] \leftarrow dist[s, v_j]+w_{j,i}$

----

# 参考资料

* [Wikipedia - Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)
* [知乎 - [最短路径问题]—Dijkstra 算法最详解](https://zhuanlan.zhihu.com/p/129373740)