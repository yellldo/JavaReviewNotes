### Bean的生命周期

- 先创建BeanDefinition

- 实例化(反射)

  > 在堆空间中申请内存，对象属性一般是默认值，调用createBeanInstance， 核心逻辑是反射

- 初始化(扩展性)

  > 初始化，对象属性赋初始值。

  - 属性赋值

    > 自定义属性赋值：调用populateBean给属性赋值。
    >
    > 容器对象属性：调用invokeAwareMethods

  - 执行前置初始化方法

    > 调用BeanPostProcessor的postProcessBeforeInitialization方法

  - 执行初始化方法（invokeInitMethods）

  - 执行后置初始化方法

    > 调用BeanPostProcessor的postProcessAfterInitialization方法

- 使用对象

- 销毁对象

## 执行初始化前置后置方法时需要判断是否对Bean对象进行扩展