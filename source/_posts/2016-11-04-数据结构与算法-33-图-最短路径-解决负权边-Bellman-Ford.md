---
title: '数据结构与算法(33): 图- 最短路径(解决负权边)Bellman-Ford'
date: 2016-11-08 15:34:00
categories:
- 数据结构与算法
tags:
- 图
- 单源最短路径
---

>Dijkstra算法虽然好，但是不能解决带负权边的图。

&emsp;&emsp;想想Floyd-Warshall和Dijkstra算法，它们都是基于这样的事实：求两点之间的最短路径，看能否可以在路径中**添加k个顶点**来缩短路径。Dijkstra则是通过对dis数组进行V(顶点个数)轮松弛，得到单源最短路径。
&emsp;&emsp;而Bellman-Ford算法求最短路径是基于这样的事实：求两点之间的最短路径，看能否可以在路径中通过**添加k条边**来缩短路径。Bellman-Ford则是通过对dis数组进行E(边个数)轮松弛，得到单源最短路径。

>Bellman-Ford算法可以用于检测是否有负权回路。

```java
    /**
     * 单源最短路径：解决负权边
     */
    public class BellmanFord {

    public static void main(String[] args) {
        int[][] edges = {{0, 1, 2}, {0, 2, 6}, {0, 3, 4}, {1, 2, 3}, {2, 3, -3}};
        int vlen = 4;
        int elen = edges.length;

        int inf = 9999;

        // 存储起点到个顶点的最短路程的“估计值”
        int[] dis = new int[vlen];

        // 初始化dis, 起始顶点为0号顶点; 初始化booked数组
        for (int i = 0; i < vlen; i++){
            dis[i] = inf;
        }
        dis[0] = 0;

        // Bellman-Ford算法核心语句
        // 进行k轮松弛，最多经过vlen-1条边求得最短路径
        for (int k = 0; k < vlen - 1; k++){
            // 用来标记本轮松弛中数组dis是否发生更新
            boolean check = false;
            for (int i = 0; i < elen; i++){
                if (dis[edges[i][1]] > dis[edges[i][0]] + edges[i][2]){
                    dis[edges[i][1]] = dis[edges[i][0]] + edges[i][2];
                    check = true;
                }
            }

            // 实际操作中，常会在未到达vlen-1轮松弛前就已经计算出最短路径
            if (!check) break;
        }

        // 检测负权回路: 再进行一次松弛, 如果还可以松弛则存在负权回路
        boolean flag = false;
        for (int i = 0; i < elen; i++) {
            if (dis[edges[i][1]] > dis[edges[i][0]] + edges[i][2]){
                flag = true;
            }
        }
        if (flag){
            System.out.println("此图含有负权回路");
        }else {
            // 输出结果
            for (int d: dis){
                System.out.print(d + " ");
            }
        }
    }
}
```

Bellman-Ford算法的队列优化：在每实施一次松弛操作后，就会有一些顶点已经求得最短路径，此后这些顶点的最短路径也不会变，这在后续松弛操作上浪费了时间，因而可以在每次松弛操作后，仅对最短路径估计值发生了变化的顶点的所有出边进行松弛操作。




