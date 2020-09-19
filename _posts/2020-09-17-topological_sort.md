---
title: "Implement topological sort"
date: 2020-09-17 20:00:00 -07:00
categories:
  - note
tags:
  - c++
  - graph
---

## Problem  
There are n nodes and the are labelled from 0 to n - 1.  
Some nodes may have directed linkage, for example, if links[i] = [ai, bi] this means ai is pointing to bi.  
Implement topological sort.  


## Method 1 - BFS  
1. Convert edges into adjacency list.  
2. Maintain a "in-degree" vector.  
3. Push all nodes with in-degree == 0 into a queue.  
4. Use BFS to traverse. The steps are follow:  
5. Push back the front of the queue into final answer.  
6. Decrement the in degree of each adjacency nodes by 1.  
7. If the in degree of each adjacency nodes == 0, push the node into the queue.  
8. If there is no more node to process.  
9. Check the size of the final answer, if the size is the number of node, return final answer.  
10. If not, there is cycle in the graph and cannot do topological sort.  
```
vector<int> TopologicalSortByBFS(int N, vector<pair<int, int>> links) {
  vector<vector<int>> adj_list(N);
  vector<int> in(N, 0);
  for (auto link : links) {
    adj_list[link.first].push_back(link.second);
    in[link.second]++;
  }
  queue<int> q;
  for (int i = 0; i < in.size(); i++) {
    if (in[i] == 0) {
      q.push(i);
    }
  }
  vector<int> ans;
  while (!q.empty()) {
    auto f = q.front();
    q.pop();
    ans.push_back(f);
    for (int i = 0; i < adj_list[f].size(); i++) {
      in[adj_list[f][i]]--;
      if (in[adj_list[f][i]] == 0) {
        q.push(adj_list[f][i]);
      }
    }
  }
  return ans;
}
```
## Method 2 - DFS  
