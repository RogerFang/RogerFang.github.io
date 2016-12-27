---
title: '数据结构与算法(31): 图- 多源最短路径Floyd-Warshall'
date: 2016-11-06 18:36:57
categories:
- 数据结构与算法
tags:
- 图
- 多源最短路径
---

&emsp;&emsp;Floyd-Warshall 算法采用动态规划方案来解决在一个**有向图** G = (V, E) 上每对顶点间的最短路径问题，即全源最短路径问题（All-Pairs Shortest Paths Problem），其中图G允许存在权值为负的边，但不存在权值为负的回路。Floyd-Warshall 算法的时间复杂度为$O(V^3)$。

&emsp;&emsp;算法的**基本思想**：求两个顶点间的最短路径，允许经过一个或多个顶点进行中转。例如，最开始只允许经过1号顶点进行中转，接下来只允许经过1号和2号顶点进行中转······允许经过1~n号所有顶点进行中转，求得任意两点之间的最短路径。
>一句话概括：从i号顶点到j号顶点只经过前k号点的最短路径。

```java
    public class FloydWarshall {

    public static void main(String[] args) {
        int[][] edges = {{0, 1, 2}, {0, 2, 6}, {0, 3, 4}, {1, 2, 3}, {2, 0, 7}, {2, 3, 1}, {3, 0, 5}, {3, 2, 12}};
        int vlen = 4;
        int elen = edges.length;

        int[][] e = new int[vlen][vlen];
        int inf = 1000;

        // 初始化 边的权重矩阵
        for (int i = 0; i < vlen; i++) {
            for (int j = 0; j < vlen; j++) {
                if (i == j){
                    e[i][j] = 0;
                }else {
                    e[i][j] = inf;
                }
            }
        }

        // 读入边
        for (int i = 0; i < elen; i++){
            e[edges[i][0]][edges[i][1]] = edges[i][2];
        }

        // Floyd-Warshall算法核心语句
        for (int k = 0; k < vlen; k++){
            for (int i = 0; i < vlen; i++){
                for (int j = 0; j < vlen; j++){
                    if (e[i][j] > e[i][k]+e[k][j]){
                        e[i][j] = e[i][k] + e[k][j];
                    }
                }
            }
        }

        // 输出结果
        for (int i = 0; i < vlen; i++){
            for (int j = 0; j < vlen; j++){
                System.out.print(e[i][j] + " ");
            }
            System.out.println();
        }
    }
}
```

测试结果：
```
0 2 5 4
9 0 3 4
6 8 0 1
5 7 10 0
```