---
title: "Implement topological sort"
date: 2020-09-17 20:00:00 -07:00
categories:
  - note
tags:
  - c++
  - graph
---

# Problem
There are n nodes and the are labelled from 0 to n - 1.
Some nodes may have directed linkage, for example, if links[i] = [ai, bi] this means ai is pointing to bi.  
Implement topological sort.  


## Method 1 - BFS
- Convert edges into adjacency list.
- Maintain a "in-degree" vector.
- Push all nodes with in-degree == 0 into a queue.
- Use BFS to traverse. The steps are follow:
- Push back the front of the queue into final answer.
- Decrement the in degree of each adjacency nodes by 1.
- If the in degree of each adjacency nodes == 0, push the node into the queue.
- If there is no more node to process.
- Check the size of the final answer, if the size is the number of node, return final answer.
- If not, there is cycle in the graph and cannot do topological sort.
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
- Convert edges into adjacency list.
- Maintain a visited list and use DFS to traverse, steps are follow:
- If node is unvisited, mark it visited. Then do the same DFS recursively to each adjacency nodes.
- After all adjacency nodes are done, push its node to the answer.
- Reverse the answer and return the answer
- We can also use an extra list to store the visited information so as to detect cycle.
```
void dfs(int node, const vector<vector<int>> &adj_list, vector<int> &ans, vector<bool> &visited) {
      if (!visited[node]) {
        visited[node] = true;
        for (int i = 0; i < adj_list[node].size(); i++) {
          dfs(adj_list[node][i], adj_list, ans, visited);
        }
        ans.push_back(node);
      }
}
vector<int> TopologicalSortByDFS(int N, vector<pair<int, int>> links) {
      vector<vector<int>> adj_list(N);
      for (auto link : links) {
        adj_list[link.first].push_back(link.second);
      }
      vector<int> ans;
      vector<bool> visited(N, false);
      for (int i = 0; i < N; i++) {
        dfs(i, adj_list, ans, visited);
      }
      reverse(ans.begin(), ans.end());
      return ans;
}
```
