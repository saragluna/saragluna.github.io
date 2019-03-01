---
layout:     post
title:      "Minimum Spanning Tree"
subtitle:   "Notes for Kruskal's algorithm, Prim's algorithm"
date:       "2019-02-20 12:00:00"
author:     "Sarag"
header-img: "img/post-bg-2019.jpg"
tags:
    - Algorithm
---



This is a note about minimum spanning tree algorithms.

## 1. Concepts

> First, think about the problem with such **assumptions**:
>
> a. edge weights are distinct
>
> b. Graph is connected
>
> Then the **Consequence** is **MST exists**

1. spanning tree of G is a subgraph T that is both a tree(connected and acyclic) and spannig(includes all of the vertices)
2. **cut**: a cut in a graph is a partition of its vertices into two (noempty) sets

2. **crossing edge**: a crossing edge connects a vertex in one set with a vertex in the other

3. **cut property**: given any cut, the crossing edge of min weight is in the MST

### Greedy MST algorithm --> easiest

To implement a MST algorithm, we should decide `how to find a cut` and `how to connect component`, classical **efficient** algorithm implementations are: 

- Kruskal's algorithm 
- Prim's algorithm 
- Borůvka's algrithm  

> Then **remove two simplifying assumptions**, the algorithm still works:
>
> - weight not distinct -> still correct. 
> - not connected -> minimum spanning forest.  



## 2. Weighted edge API

```java
public class Edge implements Comparable<Edge> {
    private final int v, w;
    private final double weight;
    
    public Edge(int v, int w, double weight) {
        this.v = v;
        this.w = w;
        this.weight = weight;
    }
    public int either() { return v; } // either endpoint
    
    public int other(int vertex) { // other endpoint
		return vertex == v ? w : v;
    }

    public int compareTo(Edge that) {
        if (this.weight < that.weight) return -1;
        else if (this.weight > that.weight) return 1;
        else return 0;
    }
}
```

### Edge-weighted graph API

```java
public class EdgeWeightedGraph {
    private final int V;
    private final List<Edge>[] adj;
    
    EdgeWeightedGraph(int V) {
        this.V = V;
        adj = new ArrayList[V];
        for (int i = 0; i < V; i++) {
            adj[i] = new ArrayList<>();
        }
    }
    // add weighted edge e to this graph 
    void addEdge(Edge e) {
		int v = e.either(), w = e.other(v);
        adj[v].add(e);
        adj[w].add(e);
    }
    // edges incident to v
    Iterable<Edge> adj(int v) {
        return adj[v];
    }
    // all edges in this graph
    Iterable<Edge> edges() {
        
    }
    // number of vertices
    int V() {
        return V;
    }
    // number of edges
    int E() {}
} 
```

### Minimum spanning tree API  

```java
public class MST {
    MST(EdgeWeightedGraph G) {}
    
    Iterable<Edge> edges() {}
    
    double weight() {} 
}
```



## 3. Kruskal's algorithm

Kruskal's algorithm is a greedy algorithm based on edge,  sort edge by weight in ascending order, then add edge to MST.

###  How to implement?

> Consider edges in ascending order of weight 
>
> - Add next edge to tree T unless doing so would create a cycle

### Proof

>**Proposition.** Kruskal's algorithm computes the MST
>
>**Pf.** Kruskal's algorithm is a special case of the greedy MST algorithm
>
>- Suppose Kruskal's algorithm colors the edge e = v-w black
>- **Cut** = set of vertices connected to v int tree T
>- No crossing edge is black => ***otherwise it would create a cycle***
>- No crossing edge has lower weight.  => ***edge are sorted in ascending orde*r**

### Challenge

`Would adding edge v-w to tree T create a cycle? If not, add it.`

- **how to detect cycle**

| O        | How?                                                         |
| -------- | ------------------------------------------------------------ |
| **V**    | run DFS from v, check if w is reacheable (T has at most V -1 edges) |
| **logV** | use the union-find data structure                            |

- use **union-find** data structure
  1. Maitain a set for each connected component in T.
  2. If v and w are in same set, then adding v-w would create a cycle
  3. To add v-w to T, merge sets containing v and w

### Java implementation

```java
public class KruskalMST {
    private Queue<Edge> mst = new Queue<>();
    
    public KruskalMST(EdgeWeightedGraph G) {
        // sort edges ascending by weight
        MinPQ<Edge> pq = new MinPQ<>();
        for (Edge e : G.edges) {
            pq.insert(e);
        }
        
        UF uf = new UF(G.V());
        // add edge to graph, check would it create a cycle
        while (!pq.isEmpty() && mst.size() < G.V() - 1) {
        	Edge e = pq.delMin();
            int v = e.either(), w = e.other(v);
            if (!uf.connected(v, w)) { // edge v-w does not create cycle
                uf.union(v, w); // merge sets
                mst.enqueue(e); // add edge to MST
            }
        }
    }
    
    Iterable<Edge> edges() {
        return mst;
    }
    
    double weight() {} 
}
```

### running time

>**Proposition.** Kruskal's algorithm computes MST in time proportional to ***ElogE*** (in the worst case)

**pf.**

| operation      | frequency | time per op |
| -------------- | --------- | ----------- |
| **build pq**   | **1**     | **E**       |
| **delete-min** | **E**     | **logE**    |
| **union**      | **V**     | **log*V**   |
| **connected**  | **E**     | **log*V**   |



## 4. Prim's algorithm

Prim's algorithm is a greedy algorithm based on vertex, adding the lowest weight vertex to the MST.

### How?

>- Start with vertex 0 and greedily grow tree T.
>- Add to T the min weight edge with exactly one endpoint in T.
>- Repeat until V-1 edges

### Proof

>**Proposition.** Prim's algorithm computes the MST.
>
>**Pf.** Prim's algorithm is a special case of the greedy MST algorithm
>
>- Suppose edge e = min weight edge connecting a vertex on the tree to a vertex not on the tree
>- Cut = set of vertices connected on tree
>- No crossing edge is black => ***because the vertex is not on the tree*** 
>- No crossing edge has lower weight => ***because we pick the minimum***

### Challenge

`find the min weight edge with exactly one endpoint in T`

- **how to find?**

  | O    | Operation                |
  | ---- | ------------------------ |
  | E    | try all edges            |
  | logE | **use a priority queue** |

### Lazy solution

#### Maintain a PQ of edges with (at least) one endpoint in T

1. Key = edge; priority = weight of edge.
2. Delete-min to determin next edge e = v-w to add to T.
3. Disregard if both endpoints v and w are in T.
4. Otherwise, let w be the vertex not in T:

- add to PQ any edge incident to w (assuming other endpoint not in T)  => ***why the algorithm is lazy***
- add w to T

#### Java implementation

```java
public class LasyPrimMST {
    private boolean[] marked;
    private Queue<Edge> mst;
    private MinPQ<Edge> pq;
    
    public LasyPrimMST(EdgeWeightedGraph G) {
        pq = new MinPQ<>();
        mst = new Queue<>();
        marked = new boolean[G.V()];
        
        visit(G, 0); // entry vertex
        
        while (!pq.isEmpty()) {
            Edge e = pq.delMin();
            int v = e.either(), w = e.other(v);
            if (marked[v] && marked[w]) continue;
            mst.enqueue(e);
            if (!marked[v]) visit(G, v);
            if (!marked[w]) visit(G, w);
        }
    }
    
    private void visit(EdgeWeightedGraph G, int v) {
        // mark vertex v 
       	marked[v] = true;
        // add edges connected to v to pq, disregard if both endpoints are marked
        for (Edge e : G.adj(v)) {
            if (!marked[w]) {
                this.pq.insert(e);
            }
        }
    }
} 
```

#### running time

>**Proposition**: Lazy Prim's algorithm computes the MST in time proportional to **ElogE** and extra space proportional to E(in worst case).

**Pf.**

| operation  | frequency | binary heap |
| ---------- | --------- | ----------- |
| delete min | E         | $logE$      |
| insert     | E         | $logE$      |

### Eager implementation

#### Maintain a PQ of vertices connected by an edge to T, where priority of vertex v = weight of shortest edge connecting v to T.

`eager means when an edge added to pq, it's for now the shortest edge connect to T `

1. Start with vertex 0 and greedily grow T
2. Add to T the min weight edge with exactly one point in T
3. Repeat until V-1 edges

#### Indexed priority queue

```java
/**
 * Associate an index between 0 and N-1 with each key in pq
 * Client can insert and delete the minimum
 * Client can change the key by specifying the index
 */
public class IndexMinPQ<Key extends Comparable<Key>> {
    IndexMinPQ(int N) {}
    void insert(int i, Key key) {}
    void decreaseKey(int i, Key key) {}    
    boolean contains(int i) {}
    int delMin() {}
    boolean isEmpty() {}
   	int size() {}
}
```

#### running time

depends on PQ implementation, V insert, V decrease min, E decrease key.

E > V

| PQ implementation | insert    | delete-min | decrease key | total                              |
| ----------------- | --------- | ---------- | ------------ | ---------------------------------- |
| array             | 1         | V          | 1            | $V^2 => (V + V^2 + E)$             |
| binary heap       | $logV$    | $logV$     | $logV$       | $ElogV => (VlogV + VlogV + ElogV)$ |
| d-way heap        | $log_d V$ | $dlog_dV$  | $log_dV$     | $Elog_1/vV$                        |
| Fibnacci heap     | 1         | $logV$     | 1            | $E + logV$                         |

**extra space is V not E**

