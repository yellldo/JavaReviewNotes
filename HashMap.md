## HashMap

### 1.8之后为什么要用红黑树

> 提高了查询效率 将原本O(n)的时间复杂度降低到了O(logn)

### 阈值为什么是0.75

> 时间和空间成本上寻求的一种折衷选择

### 为什么初始化大小是16

1. 1<<4 = 16 位运算比算数运算效率要高
2. 选16是为了服务将key映射到index算法
3. index = HashCode（Key） & （Length- 1）  
   length - 1 = 15 15的二进制是1111，这种情况下index的结果等同于HashCode后几位的值  
   为了实现均匀分布

### 链表转树的契机

1. 数组大小大于等于64，链表长度大于8