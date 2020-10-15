# LintCode Maximum Conn Area


At first, I thought this was just a simple BFS challenge with a little change. It was evaluated as a medium level; however, the process of solving problem was even interesting out of my expectation. It even helped me solve my myths about Union Find.

## Problem Description
> There is a two-dimensional array, only consists of 0 and 1. You can change a 0 to 1 at most once, please calculate the maximum area of connected 1s. If two 1s are adjcent up to down or left to right, they are regrarded connected.  

Sample input:  
```cpp
[[0,1],[1,0]]
```  
Sample output: `3`  

Notice:  
the two-dimensional array has n rows and m columns, `1 <= n, m <= 500`.  

Reference:  
LintCode 261. Maximum Connected Area

## First Solution with BFS [Fail]
In first solution, my idea was:  
1. Iterate the all points `0` (which was also `0->1`)
2. Started BFS from the point, visited neighbor who was `1`, and counted the size of island
```cpp
class Solution {
public:
    bool isValid(vector<vector<int>> &m, vector<vector<bool>> &visited, int i, int j) {
        if(i < 0 || j < 0 || i >= m.size() || j >= m[0].size())
            return false;
        if(visited[i][j])
            return false;
        if(m[i][j] == 0)
            return false;
        return true;
    }

    int onesize(vector<vector<int>> &m, vector<vector<bool>> &visited, int i, int j) {
        queue<pair<int, int>> q;
        q.push({i, j});
        visited[i][j] = true;
        
        int size = 0;
        vector<vector<int>> dir = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        while(!q.empty()) {
            int curx = q.front().first;
            int cury = q.front().second;
            q.pop();
            size++;
            for(auto d : dir) {
                int nxtx = curx + d[0];
                int nxty = cury + d[1];
                if(!isValid(m, visited, nxtx, nxty))
                    continue;
                q.push({nxtx, nxty});
                visited[nxtx][nxty] = true;
            }
        }
        
        return size;
    }

    int maxArea(vector<vector<int>> &matrix) {
        int n = matrix.size();
        int m = matrix[0].size();
        if(n == 1 && m == 1) {
            if(matrix[0][0] == 0)
                return 0;
            else
                return 1;
        }
        
        int area = 1;
        // visit from the changed point: 0->1
        vector<vector<bool>> visited(n, vector<bool>(m, false));
        
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(!visited[i][j] && matrix[i][j] == 0) {
                    area = max(area, onesize(matrix, visited, i, j));
                }
            }
        }
        
        return area;
    }
};
```
However, this idea was not correct. Assume the input,  
```cpp
[[0, 0, 1, 1, 0],
 [0, 0, 0, 1, 0],
 [1, 1,*0, 0, 1],
 [0, 0, 1, 1, 1],
 [0, 0, 1, 1, 0]]
```
and my start point was `matrix[2][2]` (the `*` one). After visiting the neighbors, points `1`s at the bottom would all show `true` in `visited[]`. But this was the point, if we start from `matrix[2][3]` in next time, we still need to visit these points `1`s; however, we can't because of `visited[]`.  

How about reseting `visited[]` in each loop? It would even cause to terrifying time complexity with BFS :sweat_smile:

## Second Solution with Union Find
In second solution, my idea worked with union find,  
1. Count the size of `1`'s islands, `father[]` showed the representive of each islands
2. Iterated the point `0` (same, which was `0->1`), and visited the neighborhood which was `1`'s islands, and sum up the size of each `1`'s islands
```cpp
class Solution {
private:
    vector<int> father;
    vector<int> area_;
    int not_zero = 0;
    vector<vector<int>> dir = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    int find(int x) {
        if(father[x] == x)
            return x;
        return father[x] = find(father[x]);
    }
    void connect(int x, int y) {
        int fx = find(x);
        int fy = find(y);
        if(fx != fy) {
            father[fx] = fy;
            area_[fy] += area_[fx];
        }
    }
public:
    inline int hash(int i, int j, int m) { return i*m + j; }
    
    bool isValid(vector<vector<int>> &m, int i, int j) {
        if(i < 0 || j < 0 || i >= m.size() || j >= m[0].size())
            return false;
        if(m[i][j] == 0)
            return false;
        return true;
    }

    int maxArea(vector<vector<int>> &matrix) {
        int n = matrix.size();
        int m = matrix[0].size();
        
        int area = 0;
        father.resize(n*m);
        area_.resize(n*m);
        
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(matrix[i][j] == 1) {
                    not_zero++;
                    father[hash(i, j, m)] = hash(i, j, m);
                    area_[hash(i, j, m)] = 1;
                }
            }
        }
        
        // exception handling
        // for all with 1
        if(not_zero == n*m)
            return not_zero;
        
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(matrix[i][j] == 1) {
                    for(auto d : dir) {
                        int nxtx = i + d[0];
                        int nxty = j + d[1];
                        if(isValid(matrix, nxtx, nxty))
                            connect(hash(i, j, m), hash(nxtx, nxty, m));
                    }
                }
            }
        }
        
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(matrix[i][j] == 0) {
                    int a = 1;
                    vector<bool> visited(n*m, false);
                    for(auto d : dir) {
                        int nxtx = i + d[0];
                        int nxty = j + d[1];
                        if(isValid(matrix, nxtx, nxty)) {
                            // be careful, don't use father[] here
                            int f = find(hash(nxtx, nxty, m));
                            if(!visited[f]) {
                                visited[f] = true;
                                a += area_[f];
                            }
                        }
                    }
                    area = max(area, a);
                }
            }
        }
        
        return area;
    }
};
```
Here are several details:  
* I used `hash()` to compress the cordinates of 2D to 1D, it would be convenient for `father[]`
* `visited[]` was the most important point of this challenge. If I put `visited[]` outside of for-loop, it mean the `1`'s islands visited before would not be visited again in future. But it doesn't make sense! Therefore, I put it into the inside of for-loop, which mean for **single** point of `0->1`, `visited[]` would not let us to add the size of duplicated `1`'s islands.

## Myth about father[] and find() in union find
So you can find there was a comment of `be careful, dont use father[]here` in here in final part. This was about my myth on union find. Take review of the work of two function for union find:  
1. `find()`: Find the root with path compression.
2. `connect()`: Merge two sets.

The key point was in `connect()`. Assume two sets with,  
```cpp
father[x] = fx;
father[y] = fy;
```
Although in part of `find()`, we ensure that all nodes point to the **root** with path compression; however, `father[fx] = fy` doesn't include path compression, which means `father[x]` was still `fx` instead of `fy`! 

Therefore, if we want to return root in future, the safest way is `find(x)` instead of `father[x]`.  

Feel free to leave any comments. I expect to get the answer with higher time complexity :blush:
