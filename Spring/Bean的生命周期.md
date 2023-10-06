### Bean的生命周期

- 先创建BeanDefinition

- 实例化(反射)

  > 在堆空间中申请内存，属性值赋默认值，调用createBeanInstance， 核心逻辑是反射

- 初始化

  - 填充属性

    > 调用populateBean给属性赋值

  - 设置容器对象属性

    > 调用invokerAwareMethods方法 **Aware接口**

  - 执行前置初始化方法

    > 调用BeanPostProcessor的postProcessBeforeInitialization方法

  - 调用invokeInitMethods方法

  - 执行后置初始化方法

    > 调用BeanPostProcessor的postProcessAfterInitialization方法

- 使用对象
- 销毁对象

## 执行初始化前置后置方法时需要判断是否对Bean对象进行扩展