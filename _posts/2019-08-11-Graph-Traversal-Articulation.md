---
layout:     post
title:      "UVa Graph Traversal - Articulation"
subtitle:   ""
date:       2019-07-29
author:     "Java"
header-img: "img/home-bg.jpg"
catalog: true
tags:
  - Uva
  - Articulation
  - Graph
---
# Graph Traversal - Articulation

## 名詞解釋
* Articulation Vertex : 讓無向圖維持連通的點,若移除此點無向圖就會分離成2個以上的子圖
* Articulation Bridge : 讓無向圖維持連通的邊,與其相接的點必為Articulation Vertex

## 題型

### 求Articulation Vertex數量 
### [UVa 315 - Network](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=251)

> 找任意點建立DFSTree,用DFSTree判斷,有兩種情形
> 1. 判斷點為根節點
> 利用DFS的特性,根節點如果有兩個以上的子樹，
> 代表子樹互相不連通,則此根節點為Articulation Vertex
> 1. 判斷點不為根節點
> 判斷點的子樹所有的Back Edge是否有深度比該點小(<=),沒有則此點為Articulation Vertex

``` java
public static void main(String[] args) throws IOException {
	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
	StringBuilder sb = new StringBuilder();
	String str;
	while (! (str = br.readLine()).equals("0")) {
		int nodeNum = Integer.parseInt(str.trim());
		boolean[][] map = new boolean[nodeNum][nodeNum];
		while (! (str = br.readLine()).equals("0")) {
			String[] line = str.split(" ");
			int node1 = Integer.parseInt(line[0]);
			for (int i = 1; i < line.length; i++) {
				int node2 = Integer.parseInt(line[i]);
				map[node1 - 1][node2 - 1] = true;
				map[node2 - 1][node1 - 1] = true;
			}
		}
		int root = 0;
		int[] visit = new int[nodeNum];
		int[] low = new int[nodeNum];
		sb.append(DFS(root, root, 0, visit, low, map) + "\n");
	}
	System.out.print(sb);
}

private static int DFS(int parent, int now, int depth, int[] visit, int[] low
        , boolean[][] map) {
	visit[now] = low[now] = ++depth;
	int ans = 0;
	int childNum = 0;
	boolean ap = false;
	for (int i = 0; i < visit.length; i++) {
		if (map[now][i] && i != parent) {
			if (visit[i] > 0) {
				low[now] = Math.min(low[now], visit[i]);
			} else {
				childNum++;
				ans += DFS(now, i, depth, visit, low, map);

				low[now] = Math.min(low[now], low[i]);
				if (low[i] >= visit[now]) ap = true;
			}
		}
	}
	if ((now == parent && childNum > 1) || (now != parent && ap)) {
		ans++;
	}
	return ans;
}
```
### 尋找所有Articulation Bridge 
### [UVa 796 - Critical Links](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=737)

> 用DFS找Articulation Vertex的原理是判斷某點的每條邊有沒有Articulation Bridge
> 如果有某點就是Articulation Vertex,所以解法跟上一題一樣
> 因為輸出要求排序,所以用一個Adjacency List紀錄Articulation Bridge

``` java
public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    String str;
    while ((str = br.readLine()) != null && str.length() != 0) {
        StringBuilder sb = new StringBuilder();
        int nodeNum = Integer.parseInt(str);
        boolean[][] map = new boolean[nodeNum][nodeNum];
        List < List < Integer >> bridges = new ArrayList < > ();
        for (int i = 0; i < nodeNum; i++) {
            bridges.add(new ArrayList < > ());
        }
        for (int i = 0; i < nodeNum; i++) {
            String[] line = br.readLine().split(" ");
            int node1 = Integer.parseInt(line[0]);
            int num = Integer.parseInt(line[1].replace("(", "").replace(")", ""));
            for (int x = 0; x < num; x++) {
                int node2 = Integer.parseInt(line[2 + x]);
                map[node1][node2] = true;
            }
        }

        int[] visit = new int[nodeNum];
        int[] low = new int[nodeNum];
        for (int i = 0; i < nodeNum; i++) {
            if (visit[i] == 0) {
                DFS(i, i, 1, visit, low, map, bridges);
            }
        }
        int bridgeNum = 0;
        for (int i = 0; i < nodeNum; i++) {
            List < Integer > nodeBridges = bridges.get(i);
            Collections.sort(nodeBridges);
            for (int end: nodeBridges) {
                sb.append(i + " - " + end + "\n");
            }
            bridgeNum += nodeBridges.size();
        }
        sb.insert(0, bridgeNum + " critical links\n");
        System.out.println(sb);
        br.readLine();
    }
}

private static void DFS(int parent, int now, int depth, int[] visit, int[] low, boolean[][] map,
    List < List < Integer >> bridges) {
    visit[now] = low[now] = depth++;
    for (int i = 0; i < visit.length; i++) {
        if (map[now][i]) {
            if (visit[i] == 0) {
                DFS(now, i, depth, visit, low, map, bridges);
                low[now] = Math.min(low[now], low[i]);
                if (low[i] > visit[now]) {
                    int min = Math.min(now, i);
                    int max = Math.max(now, i);
                    bridges.get(min).add(max);
                }
            } else if (parent != i) {
                low[now] = Math.min(low[now], low[i]);
            }
        }
    }
}
```
### 尋找所有的 Bridge-connected Component
### [UVa 10765 - Doves and bombs](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=737)

> 求如果某點消失,原本的圖會被分成幾個連通圖
> 用DFS找Articulation Bridge,找到就記錄在起點上
> 最後再全部+1把終點的數量補上
> 因為要根據連通圖數量排序所以加了Comparable的Pigeon的Class(Code變超醜zz)
> 可能可以用Tarjan's Algorithm做,但UVa分類在Articulation所以就這樣解了

``` java
static int root;

public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    String str;
    while (!(str = br.readLine()).equals("0 0")) {
        String[] line = str.split(" ");
        int nodeNum = Integer.parseInt(line[0]);
        int bumbNum = Integer.parseInt(line[1]);
        boolean[][] map = new boolean[nodeNum][nodeNum];
        while (!(str = br.readLine()).equals("-1 -1")) {
            line = str.split(" ");
            int node1 = Integer.parseInt(line[0]);
            int node2 = Integer.parseInt(line[1]);

            map[node1][node2] = map[node2][node1] = true;
        }
        int[] visit = new int[nodeNum];
        int[] low = new int[nodeNum];
        List < Pigeon > pigeons = new ArrayList < > ();
        for (int i = 0; i < nodeNum; i++) {
            Pigeon pigeon = new Pigeon(i, 0);
            pigeons.add(pigeon);
        }

        for (int i = 0; i < nodeNum; i++) {
            if (visit[i] == 0) {
                root = i;
                DFS(i, i, visit, low, pigeons, map);
            }
        }

        for (Pigeon pigeon: pigeons) {
            pigeon.value++;
        }
        Collections.sort(pigeons);

        for (int i = 0; i < bumbNum; i++) {
            System.out.println(pigeons.get(i).node + " " + pigeons.get(i).value);
        }
        System.out.println();
    }
}

private static void DFS(int parent, int now, int[] visit, int[] low, List < Pigeon > pigeons, boolean[][] map) {
    visit[now] = low[now] = visit[parent] + 1;
    int child = 0;
    for (int i = 0; i < visit.length; i++) {
        if (map[now][i]) {
            if (visit[i] == 0) {
                child++;
                DFS(now, i, visit, low, pigeons, map);
                low[now] = Math.min(low[now], low[i]);
                if (low[i] >= visit[now] && now != root) {
                    pigeons.get(now).value++;
                }
                if (root == now && child >= 2) {
                    pigeons.get(root).value++;
                }
            } else if (i != parent) {
                low[now] = Math.min(low[now], low[i]);
            }
        }
    }

}

class Pigeon implements Comparable < Pigeon > {
    public Pigeon(int i, int j) {
        node = i;
        value = j;
    }

    int node;
    int value;

    @Override
    public int compareTo(Pigeon p) {
        if (this.value == p.value)
            return this.node - p.node;
        else
            return p.value - this.value;
    }
}
```