### String

> 底层是SDS（动态字符串）

```c
struct sdshdr {
  // 记录buf数组中已使用字节的数量
  int len;
  // 记录buf数组中未使用字节的数量
  int free;
  // 字节数组，用于保存字符串
  char buf[];
}
```

#### 自动扩容
