## JVM Stack

1. Frame - 每个方法对应一个栈帧

   1. Local Variable Table （局部变量表）

   2. Operand Stack （操作数栈）

      对于long的处理（store and load），多数虚拟机的实现都是原子性的没必要加Volatile

   3. Dynamic Linking （动态链接）

   4. Return Address

      a()->b() 方法a调用了方法b，b方法的返回值放在什么地方

2. Heap

