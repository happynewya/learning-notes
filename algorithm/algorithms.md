## :evergreen_tree::deciduous_tree::palm_tree:

- 排序二叉树的中序遍历生成的是有序数组。这在很多排序二叉树中可以用到。

如：[99. Recover Binary Search Tree](https://leetcode-cn.com/problems/recover-binary-search-tree/). 莫里斯中序遍历就不看了。普通版的。

```C++
void recoverTree(TreeNode* root) { 
    stack<TreeNode *> st;
    TreeNode *p = root, *q = nullptr;
    TreeNode *x = nullptr, *y = nullptr;
    while(!st.empty() || p != nullptr){
        while(p != nullptr){
            st.push(p);
            p = p->left;
        }
        p = st.top(); st.pop();
        if(q != nullptr && q->val > p->val){
            y = p;
            x = (x == nullptr) ? q : x;
        }
        q = p;
        p = p->right;
    }
    x->val ^= y->val;
    y->val ^= x->val;
    x->val ^= y->val;
}
```

- 字符串相乘

  还是秋招留下的手艺，生成一个`string`，把`string`都`reverse`一下。一个值得注意的点：

  ```C++
  for(int i = 0; i < N; i++){
      int carry = 0;
      for(int j = 0; j < M; j++){
          prod = (num1[i] - '0') * (num2[j] - '0') + carry + result[i + j] - '0';
          carry = prod / 10;
          result[i + j] = (prod % 10) + '0';
      }
      if(carry)
          result[i + M] = carry + '0';
  }
  ```

  在第二个循环结束后，由于`carry`要加到`result`上。此时由于`k`的位置是历史最高位，也就是到目前位置循环处理过的最高位，故`result[k]`肯定是`0`。这里直接把`carry`放在第`i+M`位就可以了

- 链表的dummy节点非常好用。

  例子：[86. Partition List](https://leetcode-cn.com/problems/partition-list/)。

  ```C++
  ListNode* partition(ListNode* head, int x) {
      ListNode *h1 = new ListNode(-1);
      ListNode *h2 = new ListNode(-1);
      ListNode *p1 = h1, *p2 = h2;
      ListNode *p = head, **pp;
      while(p != nullptr){
          pp = (p->val < x) ? &p1 : &p2;
          (*pp)->next = p;
          *pp = (*pp)->next;
          p = p->next;
          (*pp)->next = nullptr;
      }
      if(p1) p1->next = h2->next;
      return (h1->next != nullptr) ? h1->next : h2->next;
  }
  ```

- LRU ：哈希双链表

  设计逻辑要点：

  1）dummy节点的head和tail

  2）插入的是（key, value）。类似于(addr, value)

  3）查询的时候把节点移到头部

  4）插入的时候，不管有没有，都要移到头部

  ```C++
  struct Node{
      int key;
      int val;
      Node* prev;
      Node* next;
      Node(): key(-1), val(-1), prev(nullptr), next(nullptr){ }
      Node(int k, int v): key(k), val(v), prev(nullptr), next(nullptr){ }
  };
  
  class LRUCache k{
  privatek:
      int sizek;
      Node *tail, *head;
      unordered_map<int, Node*> dict;
  public:
      LRUCache(int capacity) {
          this->size = capacity;
          this->head = new Node();
          this->tail = new Node();
          this->head->next = this->tail;
          this->tail->prev = this->head;
  
      }
  
      void moveTohead(int key){
          Node *p = dict[key];
          p->prev->next = p->next;
          p->next->prev = p->prev;
          p->next = head->next;
          head->next->prev = p;
          head->next = p;
          p->prev = head;
      }
      
      int get(int key) {
              if(dict.find(key) == dict.end())
          return -1;
          moveTohead(key);
          return dict[key]->val;
      }
      
      void put(int key, int value) {
          int val = value;
          if(dict.find(key) != dict.end()){
              dict[key]->val = value;
              moveTohead(key);
              return;
          } else {
              if(dict.size() == this->size){
                  Node *p = tail->prev;
                  dict.erase(p->key);
                  tail->prev = p->prev;
                  p->prev->next = tail;
                  delete p;
              }
              Node *p = new Node(key, val);
              dict[key] = p;
              p->next = head->next;
              p->next->prev = p;
              head->next = p;
              p->prev = head;
          }
  
      }
  };
  
  /**
   * Your LRUCache object will be instantiated and called as such:
   * LRUCache* obj = new LRUCache(capacity);
   * int param_1 = obj->get(key);
   * obj->put(key,value);
   */
  ```

  