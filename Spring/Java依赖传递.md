# Java依赖传递

依赖：任何一个有实际意义的应用都会由两个或者更多的类组成，这些类相互之间进行协作来完成特定的业务逻辑，因此就产生了依赖。

依赖有三种写法：构造函数传入依赖、Setter方法传入依赖和接口声明依赖对象。下面定义了车辆接口和实现，通过IDriver接口演示各种注入。

```java
// 定义车辆接口
public interface ICar {
    // 车辆能动
    void run();
}

// 定义各种车辆
public class SmallCar implements ICar {
    private String attr="smallCar";
    @Override
    public void run() {
        System.out.println(this.attr);
    }
}
public class BigCar implements ICar {
    private String attr;
    @Override
    public void run() {
        System.out.println(this.attr);
    }
}
```

### 构造方法传入依赖

```java
// 司机接口
public interface IDriver {
    public void driver();
}
// 实现类
public class Driver implements IDriver {
    private ICar car;
    // 构造函数注入
    public Driver(ICar car) {
        this.car = car;
    }

    @Override
    public void driver() {
        this.car.run();
    }
}

// 测试
import org.junit.Test;

public class TestDI {
    @Test
    public void test(){
        ICar car=new SmallCar();
        IDriver driver=new Driver(car);
        driver.driver();
    }
}
```

### Setter方法传入依赖

```java
// IDriver
public interface IDriver {
    public void driver();
    public void setCar(ICar car);
}
// Driver
public class Driver implements IDriver {
    private ICar car;
    
    @Override
    public void driver() {
        this.car.run();
    }

    @Override
    public void setCar(ICar car) {
        this.car=car;
    }
}
// test
public class TestDI {
    @Test
    public void test(){
        ICar car=new SmallCar();
        IDriver driver=new Driver();
        driver.setCar(car);
        driver.driver();
    }
}
```

### 接口声明依赖对象

```java
// IDriver
public interface IDriver {
    public void driver(ICar car);
}
// Driver
public class Driver implements IDriver {
    @Override
    public void driver(ICar car) {
        car.run();
    }
}
// test
public class TestDI {
    @Test
    public void test(){
        ICar car=new SmallCar();
        IDriver driver=new Driver();
        driver.driver(car);
    }
}
```

