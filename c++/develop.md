#### Lock

#### 计算机网络



#### C++ 知识 

- 虚函数介绍、作用 

  虚函数表、虚函数指针、动态多态、虚析构

  虚函数声明：

  ```c++
  class A{
  public:
      A();
      virtual void f(){
          cout << "In A f" << endl;
      }
  }
  ```

  在某个类定义的虚函数只对该类的派生类有效果。如果在该类的派生类中定义一个虚函数，那么这个定义对该派生类的派生类有效果。

  ```c++
  class B: public A{
  public:
      B();
      virtual void f(){//有没有virtual,对于B的虚函数表都是一样的，这个virtual只影响B的派生类
          cout << "In B f" <<endl; 
      }
  }
  ```

  

- 纯虚函数与虚函数 

    前者只声明不定义，后者必须定义 

- 函数指针

  如果在程序中定义了一个函数，那么在编译时系统就会为这个函数代码分配一段存储空间，**这段存储空间的首地址称为这个函数的地址**。而且**函数名表示的就是这个地址**。既然是地址我们就可以定义一个指针变量来存放，这个指针变量就叫作函数指针变量，简称函数指针。

  定义方法

  ```c++
  typedef return_type (* function_name)([parameters])
  int(*p)(int, int);
  ```

  一个有趣的现象

  ```c++
  void f(){
      cout << "f"<<endl;
  }
  int main(){
      f();
      (*f)();
      (*****f)();
      (***f)();
      (****f)();
      return 0;
  }
  ```

  可正常运行。f是函数的地址，应该是(dereference operator)`*`作用于函数地址无效？

#### 计算机网络 

- TCP握手、挥手，为什么三次、四次，详细 

- TCP与UDP区别

- 拥塞控制

- 输入url会发生什么，简要 

- http讲一下，详细 

- https讲一下，详细 

- 为什么安全，详细 

- 子网掩码的作用
   

####  操作系统

- 线程和进程 

- 进程通信 

    管道、消息IPC（消息队列、信号量、信号、共享内存）、socket 

- 线程通信 

    互斥锁、信号量、条件变量 

- 锁 

- 互斥锁、读写锁、自旋锁等 

- 死锁条件

  1. mutual exclusion 互斥条件
     The resources involved must be unshareable; otherwise, the processes would not be prevented from using the resource when necessary.
  2. hold and wait or partial allocation 持有并等待
     The processes must hold the resources they have already been allocated while waiting for other (requested) resources. If the process had to release its resources when a new resource or resources were requested, deadlock could not occur because the process would not prevent others from using resources that it controlled.
  3. no pre-emption 不剥夺
     The processes must not have resources taken away while that resource is being used. Otherwise, deadlock could not occur since the operating system could simply take enough resources from running processes to enable any process to finish.
  4. resource waiting or circular wait 循环等待
     A circular chain of processes, with each process holding resources which are currently being requested by the next process in the chain, cannot exist. If it does, the cycle theorem (which states that "a cycle in the resource graph is necessary for deadlock to occur") indicated that deadlock could occur.

  

####   数据库

- 索引作用、常见索引 

    快速检索，哈希、B+树 

- B+树原理和特点 

  多路平衡搜索树，讲一下树的构成和搜索顺序原理，三个特点：关键KEY数量决定叶子节点数量、非叶子节点仅有索引作用、所有叶子节点构成单向链表方便查询 

- 哈希索引和B+树区别、使用场景 

    哈希：等值查询，健冲突较少；B+树，范围搜索 
   

 

#### 各种树🌲数据结构

- B树

  B树就是二叉搜索树。如果经过平衡性优化，查找性能为$O(\log n)$。

  比二分查找数组的有点，插入删除时不需要大量移动数据。

- B-树

  多路搜索树（非二叉树）

- B+树

  B+树是B树的一种变体，有着比B树更高的查询性能。一个m阶B树具有如下特征：

  1、根节点至少有两个节点；

  2、每个中间节点都包含k-1个元素和k个孩子，其中m/2<=k<=m；

  3、每一个叶子节点都包含k-1个元素，其中m/2<=k<=m；

  4、所有的叶子节点都位于同一层；

  5、每个节点中的元素从小到大排列，节点当中k-1个元素正好是k个孩子包含的元素的值域划分。

- B+树和B树有一些共同特征，但是B+树也具备一些新的特征：

  1、有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据都保存在叶子节点；

  2、所有的叶子节点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子节点本身依关键字的大小自小而大顺序链接；

  3、所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。

综合起来，B+树比B树的优势有三个：

1、单一节点存储更多的元素，使得查询的IO次数减少；

2、所有查询都要查找到叶子节点，查询性能稳定；

3、所有叶子节点形成有序链表，便于范围查询。







  算法题1：很大量的int型数，输出前100最大的数

  算法题2：单向链表，头尾奇偶交替输出（中等） 


面经

作者：CJL1995
链接：https://www.nowcoder.com/discuss/453475?channel=-2&source_id=discuss_terminal_discuss_sim
来源：牛客网



1 const int* p与int * const p区别 

- `int const *p` and `const int *p`: `p` is a pointer to a const int.
  - reassign of the reference of `p` is allowed, but we can't change the reference number's value.
  - `p = &i;//correct`
  - `*p = 3;`//error
- `int * const p`: `p` is a const of a pointer to int.
  - can not reassign reference of `p`, but changing the referenced number's value is allowed
  - `p = &i;//error`
  - `*p = 3;//correct`
- 一个启发：
  - `int const a`和`const int b`有什么区别？No difference.

  2 指针与引用的区别（尽可能多的讲） 

[Pointers](https://www.geeksforgeeks.org/pointers-in-c-and-c-set-1-introduction-arithmetic-and-array/): A pointer is a variable that holds memory address of another variable. A pointer needs to be dereferenced with ***** operator to access the memory location it points to.

[References](https://www.geeksforgeeks.org/references-in-c/) : A reference variable is an alias, that is, another name for an already existing variable. A reference, like a pointer, is also implemented by storing the address of an object.

区别：

​	1）初始化角度。指针可以先定义，后初始化，引用需要在定义的时候同时初始化

​	2）重定义。指针可以re-assignment，引用不可以

​	3）内存。指针占用独立内存，引用和引用变量共享一个内存

​	4）空指针NULL。指针可以为空，引用不可以

​	5）间接引用。指针的指针合法，但是没有引用的引用。

  3 内存分区 、堆区和栈区的区别、特点 

  4 堆区的实现原理？不会 

  5 字节对齐的原理？有哪些方式实现字节对齐（不会）？ 

  6 HTTPS的握手流程和实现原理？加密过程？ 

  7 对称会话公匙？为什么不采用非对称的？不会，面试官解答了，说是因为对称会话密匙效率更高。 

  8 非对称会话密匙的加密原理？ 

  9 TCP的滑动窗口与拥塞控制的实现机制？ 

  10 进程与线程的区别？（讲的不够全面） 

  11协成了解吗？ 不会 

  12 进程间通信方式？（Socket必须在不同主机、不同进程之间通信吗？） 

  13 手撕代码：1 一个数n的二进制位有多少个1？写了两种算法。但是面试官要求时间复杂度O(1)的算法，不会。面试官提示了用空间换时间，提前存储每个n的1的个数。 

  2 根据前序遍历和中序遍历重建二叉树