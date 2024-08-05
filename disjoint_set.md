# 并查集

[TOC]

## 概念、原理

### 代码模板

```c++
class UnionFind {
public:
    vector<int> parents, ranks;
    int count;
    UnionFind(int n):parents(n, 0), ranks(n, 1), count(n) {
        for (int i = 0; i < n; ++i) {
            parents[i] = i;
        }
    }
    
    int Find(int x) {
        if (x != parents[x]) {
            parents[x] = Find(parents[x]);
        }
        return parents[x];
    }
    
    void Union(int x, int y) {
        int root_x = Find(x);
        int root_y = Find(y);
        if (root_x == root_y) {
            return;
        }
        if (ranks[root_x] == ranks[root_y]) {
            parents[root_x] = root_y;
            ++ranks[root_y];
        } else if (ranks[root_x] < ranks[root_y]) {
            parents[root_x] = root_y;
        } else {
            parents[root_y] = root_x;
        }
        --count;
    }
    
    bool IsConnected(int x, int y) {
        return Find(x) == Find(y);
    }
    
    int GetCount() { return count; }
};
```

## 参考链接

* https://zhuanlan.zhihu.com/p/93647900

## 题目

### [130. 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

> 给你一个 m x n 的矩阵 board ，由若干字符 'X' 和 'O' ，找到所有被 'X' 围绕的区域，并将这些区域里所有的 'O' 用 'X' 填充。
>
> 示例 1：
>
> ![img](https://assets.leetcode.com/uploads/2021/02/19/xogrid.jpg)
>
> 输入：board = [["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","X","X"]]
> 输出：[["X","X","X","X"],["X","X","X","X"],["X","X","X","X"],["X","O","X","X"]]
> 解释：被围绕的区间不会存在于边界上，换句话说，任何边界上的 'O' 都不会被填充为 'X'。 任何不在边界上，或不与边界上的 'O' 相连的 'O' 最终都会被填充为 'X'。如果两个元素在水平或垂直方向相邻，则称它们是“相连”的。
> 示例 2：
>
> 输入：board = [["X"]]
> 输出：[["X"]]

* 思路

  本题给定的矩阵中有三种元素

  * 字母`X`
  * 被字母`X`包围的字母`O`
  * 没有被字母包围的字母`O`

  本题要求将所有被字母`X`包围的字母`O`都变为字母`X`，但很难判断哪些`O`是被包围的，哪些`O`不是被包围的。

  注意题解中提示到：**任何边界上的`O`都不会被填充为`X`**。我们可以想到，所有的不被包围的`O`都直接或间接与边界上的`O`相连。我们可以利用这个性质判断`O`是否在边界上。具体的说：

  * 对于每一个边界上的`O`，我们以它为起点，标记所有与它直接或间接相连的字母`O`
  * 最后我们遍历这个矩阵，对于每一个字母：
    * 如果该字母被标记过，则该字母为没有被字母`X`包围的字母`O`，我们将其还原为字母`O`
    * 如果该字母没有被标记过，则该字母为被字母`X`包围的字母`O`，我们将其修改为字母`X`

* 深搜

  * 代码

    ```c++
    class Solution {
    public:
        void dfs(vector<vector<char>> &board, int i, int j, int rows, int cols) {
            if (i >= rows || j >= cols || i < 0 || j < 0 || board[i][j] != 'O') { return; }
            board[i][j] = '#';
            dfs(board, i - 1, j, rows, cols);
            dfs(board, i + 1, j, rows, cols);
            dfs(board, i, j - 1, rows, cols);
            dfs(board, i, j + 1, rows, cols);
        }
        void solve(vector<vector<char>> &board) {
            if (board.size() == 0 || board[0].size() == 0) { return; }
            int rows = board.size();
            int cols = board[0].size();
            
            for (int i = 0; i < rows; ++i) {
                dfs(board, i, 0, rows, cols);
                dfs(board, i, cols - 1; rows, cols);
            }
            
            for (int i = 0; i < cols; ++i) {
                dfs(board, 0, i, rows, cols);
                dfs(board, rows - 1, i, rows, cols);
            }
            
            for (int i = 0; i < rows; ++i) {
                for (int j = 0; j < cols; ++j) {
                    if (board[i][j] != '#') {
                        board[i][j] = 'X';
                    } else {
                        board[i][j] = 'O';
                    }
                }
            }
        }
    };
    ```

  * 复杂度

    * 时间复杂度：$O(n \times m)$，其中$n$和$m$分别为矩阵的行数和列数。深度优先搜索过程中，每一个点至多只会被标记一次
    * 空间复杂度：$O(n \times m)$，其中$n$和$m$分别为矩阵的行数和列数。主要为深度优先搜索的栈的开销。

* 广搜（忽略）

* 并查集

  * 思路

    所有边界上的`O`看做一个连通分量。遇到`O`就执行并查集合并操作，那么所有的`O`就会被分成两类

    * 和边界上的`O`在一个连通分量的，这些`O`保留
    * 不和边界上的`O`在一个连通分量的，这些`O`就是被包围的，替换

    需要注意两点

    * 并查集一般用一维数组记录节点，方便查找`parents`，所以这里需要将二维坐标转化为一维坐标
    * 边界上的`O`点形成的连通分量可能有多个，那么这里构造一个虚拟节点，将所有边界上的`O`组成的连通分量都和该虚拟节点连接，构成一个连通分量

  * 代码

    ```c++
    class Solution {
    public:
        void solve(vector<vector<char>> &board) {
            if (board.size() == 0 || board[0].size() == 0) { return; }
            int rows = board.size();
            int cols = board[0].size();
            UnionFind uf(rows * cols + 1);
            // 用一个虚拟节点, 边界上的O 的父节点都是这个虚拟节点
            int dummyNode = rows * cols;
            
            for (int i = 0; i < rows; ++i) {
                for (int j = 0; j < cols; ++j) {
                    // 遇到O进行并查集操作合并
                    if (board[i][j] == 'O') {
                        if (i == 0 ||i == rows - 1 || j == 0 || j == cols - 1) {
                            // 边界上的O,把它和dummyNode 合并成一个连通区域.
                            uf.Union(i * cols + j, dummyNode);
                        } else {
                            // 和上下左右合并成一个连通区域. 这个else的操作必须要有，要确保非边界上但和边界O直接或间接相连的O同处于dummyNode连通分量中，否则会出错，参考测试用例  [["O","O","O"],["O","O","O"],["O","O","O"]]
                            if (i > 0 && board[i - 1][j] == 'O') {
                                uf.Union(i * cols + j, (i - 1) * cols + j);
                            }
                            if (i < rows - 1 && board[i + 1][j] == 'O') {
                                uf.Union(i * cols + j, (i + 1) * cols + j);
                            }
                            if (j > 0 && board[i][j - 1] == 'O') {
                                uf.Union(i * cols + j, i * cols + (j - 1));
                            }
                            if (j < cols - 1 && board[i][j + 1] == 'O') {
                                uf.Union(i * cols + j, i * cols + (j + 1));
                            }
                        }
                    }
                }
            }
            
            for (int i = 0; i < rows; ++i) {
                for (int j = 0; j < cols; ++j) {
                    if (uf.IsConnected(i * cols + j, dummyNode)) {
                        // 和dummyNode 在一个连通区域的,那么就是O；
                        board[i][j] = 'O';
                    } else {
                        board[i][j] = 'X';
                    }
                }
            }
        }
    };
    ```

  * 复杂度

    * 时间复杂度：$O(mn \times \alpha(mn)$，其中$m$和$n$为矩阵的行数和列数，注意当使用路径压缩和按秩合并实现并查集时，单词操作的时间复杂度为$\alpha(mn)$，其中$\alpha(x)$为反阿克曼函数，当自变量$x$的值在人类可观测的范围内（宇宙中粒子的数量）时，函数$\alpha(x)$的值不会超过5，因此也可以看做常数时间复杂度
    * 空间复杂度：$O(mn)$，其中$m$和$n$为矩阵的行数和列数，即并查集所使用的空间

### [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

> 给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
> 岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
>
> 此外，你可以假设该网格的四条边均被水包围。
>
> 示例 1：
>
> 输入：grid = [
>   ["1","1","1","1","0"],
>   ["1","1","0","1","0"],
>   ["1","1","0","0","0"],
>   ["0","0","0","0","0"]
> ]
> 输出：1
> 示例 2：
>
> 输入：grid = [
>   ["1","1","0","0","0"],
>   ["1","1","0","0","0"],
>   ["0","0","1","0","0"],
>   ["0","0","0","1","1"]
> ]
> 输出：3

* 深搜

  * 思路

    将二维网格看成一个无向图，竖直或水平相邻的1之间有边相连。为了求出岛屿的数量，可以扫描整个二维网格，如果一个位置为1，则以其为起始节点开始进行深度优先搜索。在深度优先搜索过程中，每个搜索到的1都会重新标记为0。最终岛屿的数量就是我们进行深度优先搜索的次数

  * 代码

    ```c++
    class Solution {
    public:
        void dfs(vector<vector<char>> &grid, int row, int col) {
            int rows = grid.size(), cols = grid[0].size();
            grid[row][col] = '0';
            if (row - 1 >= 0 && grid[row - 1][col] == '1') { dfs(grid, row - 1, col); }
            if (row + 1 < rows && grid[row + 1][col] == '1') { dfs(grid, row + 1, col); }
            if (col - 1 >= 0 && grid[row][col - 1] == '1') { dfs(grid, row, col - 1); }
            if (col + 1 < cols && grid[row][col + 1] == '1') { dfs(grid, row, col + 1); }
        }
        int numIslands(vector<vector<char>> &grid) {
            int res = 0;
            if (grid.size() == 0 || grid[0].size() == 0) { return res; }
            int rows = grid.size(), cols = grid[0].size();
            for (int row = 0; row < rows; ++row) {
                for (int col = 0; col < cols; ++col) {
                    if (grid[row][col] == '1') {
                        ++res;
                        dfs(grid, row, col);
                    }
                }
            }
            return res;
        }
    };
    ```

  * 复杂度

    * 时间复杂度：$O(mn)$，其中$m$和$n$分别为行数和列数
    * 空间复杂度：$O(mn)$，其中$m$和$n$分别为行数和列数。最坏情况下，整个网络均为陆地，深度优先搜索的深度达到$mn$

* 广搜

  * 思路

    为了求出岛屿数量，扫描整个二维网格。如果一个位置为1，则将其加入队列，开始进行广度优先搜索。在广度优先搜索过程中，每个搜索到的1都会被重新标记为0。直到队列为空，搜索结束。最终岛屿的数量就是我们进行广搜的次数。

  * 代码

    ```c++
    class Solution {
    public:
        int numIslands(vector<vector<char>> &grid) {
            int res = 0;
            if (grid.size() == 0 || grid[0].size() == 0) { return res; }
            int rows = grid.size(), cols = grid[0].size();
            queue<pair<int, int>> q;
            for (int row = 0; row < rows; ++row) {
                for (int col = 0; col < cols; ++col) {
                    if (grid[row][col] == '1') {
                        q.push({row, col});
                        while (!q.empty()) {
                            auto cur = q.front();
                            q.pop();
                            int r = cur.first, c = cur.second;
                            if (r - 1 >= 0 && grid[r - 1][c] == '1') {
                                q.push({r - 1, c});
                                grid[r - 1][c] = '0';
                            }
                            if (r + 1 < rows && grid[r + 1][c] == '1') {
                                q.push({r + 1, c});
                                grid[r + 1][c] = '0';
                            }
                            if (c - 1 >= 0 && grid[r][c - 1] == '1') {
                                q.push({r, c - 1});
                                grid[r][c - 1] = '0';
                            }
                            if (c + 1 < cols && grid[r][c + 1] == '1') {
                                q.push({r, c + 1});
                                grid[r][c + 1] = '0';
                            }
                        }
                      ++res;
                    }
                }
            }
            return res;
        }
    };
    ```

  * 复杂度

    * 时间复杂度：$O(mn)$，其中$m$和$n$分别为行数和列数
    * 空间复杂度：$O(min(m, n))$，其中$m$和$n$分别为行数和列数。最坏情况下，整个网络均为陆地，队列的大小可以达到$min(m, n)$

* 并查集

  * 思路

    为了求出岛屿的数量，扫描整个二维网格，如果一个位置为1，则将其与相邻四个方向上的1在并查集中进行合并，最终岛屿的数量就是并查集中连通分量的数目

  * 代码

    ```c++
    // 并查集实现采用上面的模板代码
    class Solution {
    public:
        int numIslands(vector<vector<char>> &grid) {
            int res = 0;
            if (grid.size() == 0 || grid[0].size() == 0) { return res; }
            int rows = grid.size(), cols = grid[0].size();
            UnionFind uf(rows * cols);
            int water_counts = 0;
            for (int row = 0; row < rows; ++row) {
                for (int col = 0; col < cols; ++col) {
                    if (grid[i][j] == '0') { 
                        ++water_counts; 
                    } else {
                        if (row - 1 >= 0 && grid[row - 1][col] == '1') {
                            uf.Union(row * cols + col, (row - 1) * cols + col);
                        }
                        if (row + 1 < rows && grid[row + 1][col] == '1') { 
                            uf.Union(row * cols + col, (row + 1) * cols + col);
                        }
                        if (col - 1 >= 0 && grid[row][col - 1] == '1') { 
                            uf.Union(row * cols + col, row * cols + (col  - 1));
                        }
                        if (col + 1 < cols && grid[row][col + 1] == '1') { 
                            uf.Union(row * cols + col, row * cols + (col  + 1));
                        }
                    }
                }
            }
            return uf.GetCount() - water_counts;
        }
    };
    ```

  * 复杂度

    * 时间复杂度：$O(mn \times \alpha(mn))$，其中$m$和$n$分别为行数和列数，注意当使用路径压缩和按秩合并实现并查集时，单词操作的时间复杂度为$\alpha(mn)$，其中$\alpha(x)$为反阿克曼函数，当自变量$x$的值在人类可观测的范围内（宇宙中粒子的数量）时，函数$\alpha(x)$的值不会超过5，因此也可以看做常数时间复杂度空间复杂度：$O(mn)$，其中$m$和$n$为矩阵的行数和列数，即并查集所使用的空间

### [399. 除法求值](https://leetcode-cn.com/problems/evaluate-division/)

>给你一个变量对数组 equations 和一个实数值数组 values 作为已知条件，其中 equations[i] = $[A_i, B_i]$ 和 values[i] 共同表示等式 $A_i / B_i$ = values[i] 。每个 $A_i$ 或 $B_i$ 是一个表示单个变量的字符串。
>
>另有一些以数组 queries 表示的问题，其中 queries[j] = $[C_j, D_j]$ 表示第 j 个问题，请你根据已知条件找出 $C_j / D_j = ?$ 的结果作为答案。
>
>返回 所有问题的答案 。如果存在某个无法确定的答案，则用 -1.0 替代这个答案。如果问题中出现了给定的已知条件中没有出现的字符串，也需要用 -1.0 替代这个答案。
>
>注意：输入总是有效的。你可以假设除法运算中不会出现除数为 0 的情况，且不存在任何矛盾的结果。
>
>示例 1：
>
>输入：equations = [["a","b"],["b","c"]], values = [2.0,3.0], queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]
>输出：[6.00000,0.50000,-1.00000,1.00000,-1.00000]
>解释：
>条件：a / b = 2.0, b / c = 3.0
>问题：a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ?
>结果：[6.0, 0.5, -1.0, 1.0, -1.0 ]
>示例 2：
>
>输入：equations = [["a","b"],["b","c"],["bc","cd"]], values = [1.5,2.5,5.0], queries = [["a","c"],["c","b"],["bc","cd"],["cd","bc"]]
>输出：[3.75000,0.40000,5.00000,0.20000]
>示例 3：
>
>输入：equations = [["a","b"]], values = [0.5], queries = [["a","b"],["b","a"],["a","c"],["x","y"]]
>输出：[0.50000,2.00000,-1.00000,-1.00000]

* 深搜

  * 思路

    按边思考，根据示例，如果$\frac{a}{b} = 2.0$，则把`a`和`b`相连，权值为2。反过来，`b->a`权值为$\frac{b}{a} = \frac{1}{2}$；如果要求$\frac{a}{c}$，经过的路径为`a->b->c`，权值分别为`2.0`和`3.0`，则结果就是把路径上的权值相乘

  * 代码

    ```c++
    class Solution {
    public:
        vector<double> res;
        bool NoFound;
        
        void dfs(unordered_map<string, vector<pair<string, double>>> &g, unordered_map<string, int> &visit, string start, const string &target, double path) {
            // 如果节点已经相连，则结束dfs搜索，直接返回
            if (NoFound == false) {
                return;
            }
            if (start == target) {
                NoFound = false;
                res.push_back(path);
                return;
            }
            for (int j = 0; j < g[start].size(); ++j) {
                if (visit[g[start][j].first] == 0) {
                    visit[g[start][j].first] = 1;
                    dfs(g, visit, g[start][j].first, target, path * g[start][j].second);
                    visit[g[start][j].first] = 0;
                }
            }
        }
        
        vector<double> calcEquation(vector<vector<string>> &equations, vector<double> &values, vector<vector<string>> &queries) {
            unordered_map<string, vector<pair<string, double>>> g;
            unordered_map<string, int> visit;
            
            // 构建无向图（双向），a->b = 3.0，则b->a = 1 / 3
            for (int i = 0; i < equations.size(); ++i) {
                g[equations[i][0]].push_back(make_pair(equations[i][1], values[i]));
                g[equations[i][1]].push_back(make_pair(equations[i][0], 1.0 / values[i]));
            }
            
            // 遍历queries，对每一组进行dfs计算结果；如果相连接，则把路径上的权值相乘就是结果
            for (int i = 0; i < queries.size(); ++i) {
                if (g.find(queries[i][0]) == g.end()) {
                    res.push_back(-1.0);
                    continue;
                }
                
                // 全局变量，进行dfs后，若queries[i][0]无法到达queries[i][1]，则NoFound=true
                NoFound = true;
                visit[queries[i][0]]= 1;
                dfs(g, visit, queries[i][0], queries[i][1], 1.0);
                visit[queries[i][0]] = 0;
                
                if (NoFound) {
                    res.push_back(-1.0);
                }
            }
            return res;
        }
    };
    ```

  * 复杂度

* 广搜：略

* 并查集

  * 思路

    * 根据示例发现，可以将题目给出的`equation`中的两个变量所在集合进行合并，**则同在一个集合中的两个变量就可以铜鼓某种方式计算比值，即不同变量间的比值转换为相同变量的比值**

    * **构建有向图**：根据示例发现，`equations`和`values`可表示成一个图，`equations`中变量是图的定点，`values`的值是定点间**有向边**的权值，具体示例可看下图

      <img src="https://pic.leetcode-cn.com/1609860627-dZoDYx-image.png" alt="img" style="zoom: 33%;" />

    * **路径压缩**：避免并查集所表示的树形结构高度过高，影响查询性能。路径压缩就是针对树高度的优化，即在查询一个节点`a`的根节点时，把节点`a`到根节点的沿途所有节点的父节点都指向根节点，这样树的高度为2，查询性能最好。如下图示例

      <img src="https://pic.leetcode-cn.com/1609861184-fXdaCo-image.png" alt="image.png" style="zoom:33%;" />

    * **如何在*查询操作*的*路径压缩*优化中维护权值变化**：如下图所示，在查询节点`a`时，路径压缩会一层层向上找，直到很节点`d`，然后依次把`c, b, a`的父节点指向根节点

      * `c`的父节点已经是根节点，则权值不用改变

      * `b`的父节点要修改成根节点，则权值是从 当前节点到根节点经过的所有有向边的权值的乘积，为$3.0 \times 4.0 = 12.0$

      * `a`的父节点要修改为根节点，权值依然是从当前节点到根节点经过的所有有向边的权值的乘积，但要注意 **此时b的父节点已经修改为根节点d，则当前a到根节点路径为a->b->d**，故a到根节点的权值为a->b的权值乘上b->d的权值即可，$2.0 * 12.0 = 24.0$

        <img src="https://pic.leetcode-cn.com/1609861645-DbxMDs-image.png" alt="image.png" style="zoom: 25%;" />

    * **如何在*合并*操作中维护权值的变化**：要注意，合并操作时，当前的树高度最多为2，即都已完成了路径压缩，也就是 *要合并的两棵树的叶子结点到根节点最多只有一条有向边*

      如下图所示，$\frac{a}{b}=3.0, \frac{d}{c}=4.0, \frac{a}{d} = 6.0$，现在要合并`a`节点集合和`d`节点集合，就是把`a`的根节点`b`指向`d`的根节点`c`，那么如何计算`b`节点到`c`节点的有向边的权重呢？

      要注意，合并后，`a`到根节点有两条路径，分别为`a->b->c`和`a->d->c`，并且注意 **两条路径上有向边的权值乘积一定相等**，故$\frac{b}{c} = \frac{6.0 \times 4.0}{3.0} = 8.0$

      <img src="https://pic.leetcode-cn.com/1609862151-XZgKGY-image.png" alt="image.png" style="zoom: 50%;" />

  * 代码

    ```c++
    class UnionFind {
    public:
        vector<int> parents;
        vector<double> weights;
        
        UnionFind(int n): parents(n, 0), weights(n, 1.0) {
            for (int i = 0; i < n; ++i) {
                parents[i] = i;
            }
        }
        
        int Find(int x) {
            if (x != parents[x]) {
                int old_parent = parents[x];
                parents[x] = Find(parents[x]);
                weights[x] *= weights[old_parent];
            }
            return parents[x];
        }
        
        void Union(int x, int y, double v) {
            int root_x = Find(x);
            int root_y = Find(y);
            if (root_x == root_y) {
                return;
            }
            parents[root_x] = root_y;
            weights[root_x] = v * weights[y] / weights[x];
        }
        
        double IsConnected(int x, int y) {
            int root_x = Find(x);
            int root_y = Find(y);
            if (root_x == root_y) {
                return weights[x] / weights[y];
            }
            return -1.0; 
        }
    };
    
    class Solution {
    public:
        vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
            unordered_map<string, int> lookup;
            int idx = 0;
            int n = equations.size();
            vector<double> res;
            UnionFind uf(n * 2);
            
            for (int i = 0; i < n; ++i) {
                string x = equations[i][0];
                string y = equations[i][1];
                double v = values[i];
                if (lookup.find(x) == lookup.end()) {
                    lookup[x] = idx++;
                }
                if (lookup.find(y) == lookup.end()) {
                    lookup[y] = idx++;
                }
                uf.Union(lookup[x], lookup[y], v);
            }
            for (int i = 0; i < queries.size(); ++i) {
                string x = queries[i][0];
                string y = queries[i][1];
                auto x_it = lookup.find(x);
                auto y_it = lookup.find(y);
                if (x_it == lookup.end() || y_it == lookup.end()) {
                    res.push_back(-1.0);
                } else {
                    res.push_back(uf.IsConnected(x_it->second, y_it->second));
                }
            }
            return res;
        }
    };
    ```

  * 复杂度

    * 时间复杂度 $O((N + Q)logA)$
      * 构建并查集$O(NlogA)$，这里$N$为输入方程`equations`的长度，每次执行合并操作的时间复杂度都是$O(logA)$，$A$是`equations`里不同字符的个数
      * 查询并查集$O(QlogA)$，这里$Q$是查询数组`queries`的长度，每次查询执行*路径压缩*的时间复杂度是$O(logA)$
    * 空间复杂度 $O(A)$：创建字符与`id`的对应关系`unorderd_map`长度为$A$，并查集底层使用的两个数组`parent`和`weight`存储每个变量的连通分量信息，`parent`和`weight`的长度均为$A$

  * 参考 

    * [399. 除法求值](https://leetcode-cn.com/problems/evaluate-division/solution/399-chu-fa-qiu-zhi-nan-du-zhong-deng-286-w45d/)
    * [DFS详解，剖析过程，其实没那么难](https://leetcode-cn.com/problems/evaluate-division/solution/dfsxiang-jie-pou-xi-guo-cheng-qi-shi-hen-aqin/)

### [547. 省份数量](https://leetcode-cn.com/problems/number-of-provinces/) 

> 难度：中等
>
> 有 `n` 个城市，其中一些彼此相连，另一些没有相连。如果城市 `a` 与城市 `b` 直接相连，且城市 `b` 与城市 `c` 直接相连，那么城市 `a` 与城市 `c` 间接相连。
>
> **省份** 是一组直接或间接相连的城市，组内不含其他没有相连的城市。
>
> 给你一个 `n x n` 的矩阵 `isConnected` ，其中 `isConnected[i][j] = 1` 表示第 `i` 个城市和第 `j` 个城市直接相连，而 `isConnected[i][j] = 0` 表示二者不直接相连。
>
> 返回矩阵中 **省份** 的数量。

* 深搜

  * 思路

    遍历所有城市，对于每个城市，如果该城市尚未被访问过，则从该城市开始深度优先搜索，通过矩阵`isConnected`得到与该城市直接相连的城市有哪些，这些城市和该城市同属一个连通分量，然后对这些城市继续深度优先搜索，直到同一个连通分量的所有城市都被访问到，即可得到一个省份。遍历完全部城市后，即可得到连通分量的总数，即省份总数

  * 代码

    ```c++
    void dfs(vector<vector<int>> &isConnected, vector<int> &visited, int provinces, int i) {
        for (int j = 0; j < provinces; ++j) {
            if (isConnected[i][j] == 1 && !visited[j]) {
                visited[j] = 1;
                dfs(isConnected, visited, provinces, j);
            }
        }
    }
    int findCircleNum(vector<vector<int>> &isConnected) {
        int provinces = isConnected.size();
        vector<int> visited(provinces);
        int circles = 0;
        for (int i = 0; i < provinces; ++i) {
            if (!visited[i]) {
                dfs(isConnected, visited, provinces, i);
                ++circles;
            }
        }
        return circles;
    }
    ```

  * 复杂度

    * 时间复杂度 $ O(n^2) $，其中$n$是城市的数量。需要遍历矩阵$n$中的每个元素
    * 空间复杂度$O(n)$，需要使用数组$visited$记录每个城市是否被访问过，数组长度是$n$，递归调用栈的深度不会超过$n$

* 广搜

  * 思路

    对于每个城市，如果该城市尚未被访问过，则从该城市开始广度优先搜索，直到同一个连通分量中的所有城市都被访问到，即可得到一个省份

  * 代码

    ```c++
    int findCircleNum(vector<vector<int>> &isConnected) {
        int provinces = isConnected.size();
        vector<int> visited(provinces);
        int circles = 0;
        queue<int> q;
        for (int i = 0; i < provinces; ++i) {
            if (!visited[i]) {
                q.push(i);
                while (!q.empty()) {
                    int j = q.front();
                    q.pop();
                    visited[j] = 1;
                    for (int k = 0; k < provinces; ++k) {
                        if (isConnected[j][k] == 1 && !visited[k]) {
                            p.push(k);
                        }
                    }
                }
                ++circlrs;
            }
        }
        return circles;
    }
    ```

  * 复杂度

    * 时间复杂度$O(n^2)$, 其中$n$是城市的数量，需要遍历矩阵$isConnected$中的每个元素
    * 空间复杂度$O(n)$，其中$n$是城市的数量。需要使用数组$visited$记录每个城市是否被访问过，数组长度为$n$，广度优先搜索使用的队列的元素个数不会超过$n$

* 并查集

  * 思路

    初始时，每个城市都属于不同的连通分量。遍历矩阵$isConnected$，如果两个城市之间有相连关系，则他们属于同一个连通分量，对它们进行合并。遍历矩阵$isConnected$的全部元素之后，计算连通分量的总数，即为省份的总数。

  * 代码

    ```c++
    // 并查集实现采用上面的模板代码
    class Solution {
    public:
        int findCircleNum(vector<vector<int>> &isConnected) {
            int provinces = isConnected.size();
            UnionFind uf(provinces);
            for (int i = 0; i < provinces; ++i) {
                for (int j = i + 1; j < provinces; ++j) {
                    if (isConnected[i][j] == 1) {
                        uf.Union(i, j);
                    }
                }
            }
            return uf.GetCount();
        }
    };
    ```

    

  * 复杂度

    * 时间复杂度$O(n^2 logn)$,其中$n$是城市数量，需要遍历完矩阵$isConnected$的所有元素，时间复杂度是$O(n^2)$，如果遇到相连关系，则需要进行2次查找和最多一个合并，一共需要进行$2n^2$次查找和最多$n^2$次合并，因此总时间复杂度是$O(n^2log{n^2}) = O(n^2logn)$。**这里记得计算下在包含路径压缩和按秩合并情况下的时间复杂度，现在还不理解？**
    * 空间复杂度：$O(n)$，其中$n$是城市数量，需要使用到数组$parent$记录每个城市所属的连通分量的祖先

### [990. 等式方程的可满足性](https://leetcode-cn.com/problems/satisfiability-of-equality-equations/)

> 给定一个由表示变量之间关系的字符串方程组成的数组，每个字符串方程 equations[i] 的长度为 4，并采用两种不同的形式之一："a==b" 或 "a!=b"。在这里，a 和 b 是小写字母（不一定不同），表示单字母变量名。
>
> 只有当可以将整数分配给变量名，以便满足所有给定的方程时才返回 true，否则返回 false。 
>
> 示例 1：
>
> 输入：["a==b","b!=a"]
> 输出：false
> 解释：如果我们指定，a = 1 且 b = 1，那么可以满足第一个方程，但无法满足第二个方程。没有办法分配变量同时满足这两个方程。
> 示例 2：
>
> 输入：["b==a","a==b"]
> 输出：true
> 解释：我们可以指定 a = 1 且 b = 1 以满足满足这两个方程

* 并查集

  * 思路

    将每个变量看做图中的一个节点，把相等的关系`==`看作是连接两个节点的边，那么由于表示相等关系的等式方程具有传递性，即如果`a==b`和`b==c`成立，则`a==c`也成立。也就是说，所有相等的变量属于同一个连通分量。因此，我们可以使用并查集来维护这种连通分量的关系。

    首先遍历所有等式，构造并查集。同一个等式中的两个变量属于同一个连通分量，因此将两个变量进行合并。

    然后变量所有不等式。同一个不等式中的两个变量不能属于同一个连通分量，因此对两个变量分别查找其所在的连通分量，如果两个变量在同一个连通分量中，则产生矛盾，返回`false`。如果变量所有不等式没有发现矛盾，则返回`true`。

  * 代码

    ```c++
    // 并查集实现采用上面的模板代码
    class Solution {
    public:
        bool equationsPossible(vector<string> &equations) {
            unordered_map<char, int> lookup;
            UnionFind uf(equations.size() * 2);
            int idx = 0;
            for (auto &equation : equations) {
                if (lookup.find(equation[0]) == lookup.end()) {
                    lookup[equation[0]] = idx++;
                }
                if (lookup.find(equation[3]) == lookup.end()) {
                    lookup[equation[3]] = idx++;
                }
                if (equation[1] == '=') {
                    uf.Union(lookup[equation[0]], lookup[equation[3]]);
                }
            }
            
            for (auto &equation : equations) {
                if (equation[1] == '!' && uf.isConnected(lookup[equation[0]], lookup[equation[3]])) {
                    return false;
                }
            }
            return true;
        }
    };
    ```




