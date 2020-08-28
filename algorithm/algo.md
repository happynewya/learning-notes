#### DFS template

```c++
vector<vector<int>> ans;
vector<int> temp;
void dfs(int cur, vector<int>& nums) {
    if (cur == nums.size()) {
        // 判断是否合法，如果合法判断是否重复，将满足条件的加入答案
        if (isValid() && notVisited()) {
            ans.push_back(temp);
        }
        return;
    }
    // 如果选择当前元素
    temp.push_back(nums[cur]);
    dfs(cur + 1, nums);
    temp.pop_back();
    // 如果不选择当前元素
    dfs(cur + 1, nums);
}
```

对于特定的条件，考虑优化如何选择当前元素。比如引入判断条件，是否**选择**当前元素或者**不选择**当前元素。例子：Leetcode 491 increasing subsequence

```c++
 void helper(vector<vector<int>> &result, vector<int> &nums, vector<int> &cur, int i, int prev){
    if(i == nums.size()){
        if(cur.size() >= 2) 
            result.push_back(cur);
        return;
    }
    if(nums[i] >= prev) {
        cur.push_back(nums[i]);
        helper(result, nums, cur, i + 1, nums[i]);
        cur.pop_back();
    }
    if(nums[i] != prev){
        helper(result, nums, cur, i + 1, prev);
    }
}
```



#### Hierholzer Algorithm

Find Euler Path in a connected graph, the algorithm:

1) start from beginning node, do DFS

2) delete the path after moving on the path

3) if no where to go, add the node to the stack

4) pop the stack, and the order of the popping is the answer

```c++
void dfs(Node *cur) {
	while(has some where to go) {
		Node *next_node = get_next_node(cur);
		delete_path(cur, next_node);
		dfs(next_node);
	}
    result.push_back(cur);
}
```

