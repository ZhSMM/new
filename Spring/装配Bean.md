# 装配Bean

## 声明Bean

## 构造器注入和Setter方法注入

## 装配Bean

装配：创建应用对象之间协作关系的行为通常称为装配（wiring），这也是依赖注入 （DI）的本质。

装配Spring中装配Bean的方法：

- 在xml中显式配置；
- 在Java中进行显式配置；
- 隐式的bean发现机制和自动装配；

装配选择建议：

- 尽可能使用自动配置的机制；
- 使用JavaConfig配置（类型安全）；
- xml配置。

### 自动化装配

两个角度：

- 组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean；
- 自动装配（autowiring）：Spring自动满足bean之间的依赖。

#### 创建可被发现的bean





## 控制Bean的创建和销毁