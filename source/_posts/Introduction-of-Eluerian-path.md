---
title: Introduction of Eulerian path
date: 2022-02-19 10:05:51
tags: Algorithms
cover: https://s3.amazonaws.com/images.masen.com/2022/02/87f6ca33c86fb9d6a396acfb93bcaadb.png
---

# Definition

---

An **Eulerian trial** is a trial in a finite graph that visits every edge exactly once. 

An **Eulerian circuit** or **Eulerian cycle** is an Eulerian trail that starts and ends on the same vertex.

<!--more-->

# Theorems and Corollaries

---

## For Undirected Graph G

![速绘 2](https://s3.amazonaws.com/images.masen.com/2022/02/cc7e2f2adcd3a4dfcf44815efd78e45b.png)

**The sufficient requisite that undirected graph G exists Eulerian trial is that:**

- G is a connected graph
- Only exist 2 (Eulerian trial) or 0 (Eulerian circuit) vertexes have odd degrees.

Corollaries:

(1) When only existing 2 vertexes, these 2 vertexes must be the endpoints of the Eulerian trial.

(2) The sufficient requisite that graph G contains an Eulerian cycle is that there is no vertex having odd degrees.

## For Directed Graph G

![速绘 3](https://s3.amazonaws.com/images.masen.com/2022/02/4f4523e26c00bd9508caba7dab496e16.png)

The sufficient requisite that directed graph G exists Eulerian trial is that:

- The base map of graph G is connected.
- At most one vertex has (out-degree)-(in-degree) = 1, at most one vertex has (in-degree)-(out-degree) = 1, every other vertex has equal in-degree and out-degree.

Corollaries:

(1) The start point is the vertex with (out-degree)-(in-degree) = 1. The end point is the vertex with (in-degree)-(out-degree) = 1.

(2) Directed graph G exists Eulerian circle if and only if every vertex has equal in-degree and out-degree.

# How to find the Eulerian trial

---

## DFS

1. Determine the existence of the Eulerian trial of graph G using the Eulerian theorem, and find the start point and the end point as mentioned above.

2. Using DFS to iterate over every vertex ( every vertex is traversed once ). Record the traversed vertex when searching. Finally, we could get the Eulerian trial of graph G.

```c++
#include <iostream>
#include <algorithm>
#include <vector>
#include <unordered_map>

using namespace std;

void dfs(unordered_map<int, vector<int>>& neibors, int cur, vector<int>& ans, vector<bool>& visited) {
    for (int neibor: neibors[cur]) {
        if (!visited[neibor]) {
            visited[neibor] = true;
            dfs(neibors, neibor, ans, visited);
        }
    }
    ans.push_back(cur);
}

/*
 vertexes: n x 2 matrix, representing vertexes in graph G
 min: minimum value of the node
 max: maximum value of the node
 */
vector<int> EulerianTrial(vector<vector<int>>& vertexes, int min, int max) {
    int n = max-min+1;
    unordered_map<int, vector<int>> neibors; // Adjacency list
    vector<int> in(n), out(n);   // record in degree and out degree
    vector<bool> visited(n, false);
    
    for (auto& vertex:vertexes) {
        neibors[vertex[0]].push_back(vertex[1]);
        in[vertex[1]]++;
        out[vertex[0]]++;
    }
    
    bool have_fixed_start_point = false;
    vector<int> ans;
    for (int i=min; i<=max; i++) {
        if (out[i]-in[i]==1) {
            have_fixed_start_point = true;
            dfs(neibors, i, ans, visited); // exist vertex with odd in-degree, which is the start point
            break;
        }
    }
    if (!have_fixed_start_point) {
        dfs(neibors, min, ans, visited);
    }
    reverse(ans.begin(), ans.end());
    return ans;
}
```

# Application

---

A list of sticks with color on either end (ex. (cyan, violet)), and write an algorithm to find a sequence where each end of the stick is lined up with their corresponding color (ex. (cyan, violet) - (violet, red) - (red, blue)). Return the sequence if it exists, otherwise, return "impossible".

**Analysis:**

We could treat the sticks as vertexes in graph G. The corresponding color is the nodes in graph G. Then the question could be transformed to "Please find the Eulerian trial in this graph".

**Code:**

```c++
int N = 2000, Min, Max;
vector<int> visited(N);
vector<int> ans;

void dfs(vector<vector<int>>& mp, int cur) {
    for (int i=Min; i<=Max; i++) {
        if (mp[cur][i]) {
            mp[cur][i]--;
            mp[i][cur]--;
            dfs(mp, i);
        }
    }
    ans.push_back(cur);
}

void getSticks(vector<vector<int>>& sticks) {
    vector<vector<int>> mp(N, vector<int>(N, 0));
    Min = INT_MAX;
    Max = INT_MIN;
    vector<int> degrees(N);
    for (auto stick: sticks) {
        mp[stick[0]][stick[1]]++;
        mp[stick[1]][stick[0]]++;
        Min = min(Min, min(stick[0], stick[1]));
        Max = max(Max, max(stick[0], stick[1]));
        degrees[stick[0]]++;
        degrees[stick[1]]++;
    }
    
    bool flag = false;
    for (int i=Min; i<=Max; i++) {
        if (degrees[i]%2) {
            flag = true;
            dfs(mp, i);
            break;
        }
    }
    if (!flag) {
        dfs(mp, Min);
    }
    reverse(ans.begin(), ans.end());
}


int main() {    
    vector<vector<int>> sticks = {{1,2}, {2,1}, {1,3}, {3,4}};
    
    getSticks(sticks);
    
    for (int i: ans) {
        cout << i << " ";
    }
    cout << endl;
}
```

