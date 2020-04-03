# lombok入门

lombok：[官方文档](https://projectlombok.org/features/all)

lombok使用注解，来提高开发效率。

引入maven依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

常用注解：

1. @Data：注解在类上，会为类的所有属性自动生成setter/getter、equals、canEqual、hashCode、toString方法，如为final属性，则不会为该属性生成setter方法。

   官方示例：

   ```java
   @Data public class DataExample {
       private final String name;
       @Setter(AccessLevel.PACKAGE) private int age;
       private double score;
       private String[] tags;
   
       @ToString(includeFieldNames=true)
       @Data(staticConstructor="of")
       public static class Exercise<T> {
           private final String name;
           private final T value;
       }
   }
   ```

   不使用Lombok，实现：

   ```java
   public class DataExample {
       private final String name;
       private int age;
       private double score;
       private String[] tags;
   
       public DataExample(String name) {
           this.name = name;
       }
   
       public String getName() {
           return this.name;
       }
   
       void setAge(int age) {
           this.age = age;
       }
   
       public int getAge() {
           return this.age;
       }
   
       public void setScore(double score) {
           this.score = score;
       }
   
       public double getScore() {
           return this.score;
       }
   
       public String[] getTags() {
           return this.tags;
       }
   
       public void setTags(String[] tags) {
           this.tags = tags;
       }
   
       @Override public String toString() {
           return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags()) + ")";
       }
   
       protected boolean canEqual(Object other) {
           return other instanceof DataExample;
       }
   
       @Override public boolean equals(Object o) {
           if (o == this) return true;
           if (!(o instanceof DataExample)) return false;
           DataExample other = (DataExample) o;
           if (!other.canEqual((Object)this)) return false;
           if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
           if (this.getAge() != other.getAge()) return false;
           if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
           if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
           return true;
       }
   
       @Override public int hashCode() {
           final int PRIME = 59;
           int result = 1;
           final long temp1 = Double.doubleToLongBits(this.getScore());
           result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
           result = (result*PRIME) + this.getAge();
           result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
           result = (result*PRIME) + Arrays.deepHashCode(this.getTags());
           return result;
       }
   
       public static class Exercise<T> {
           private final String name;
           private final T value;
   
           private Exercise(String name, T value) {
               this.name = name;
               this.value = value;
           }
   
           public static <T> Exercise<T> of(String name, T value) {
               return new Exercise<T>(name, value);
           }
   
           public String getName() {
               return this.name;
           }
   
           public T getValue() {
               return this.value;
           }
   
           @Override public String toString() {
               return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
           }
   
           protected boolean canEqual(Object other) {
               return other instanceof Exercise;
           }
   
           @Override public boolean equals(Object o) {
               if (o == this) return true;
               if (!(o instanceof Exercise)) return false;
               Exercise<?> other = (Exercise<?>) o;
               if (!other.canEqual((Object)this)) return false;
               if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName())) return false;
               if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue())) return false;
               return true;
           }
   
           @Override public int hashCode() {
               final int PRIME = 59;
               int result = 1;
               result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
               result = (result*PRIME) + (this.getValue() == null ? 43 : this.getValue().hashCode());
               return result;
           }
       }
   }
   ```

2. @Getter/@Setter：此注解在属性上，可以为相应的属性自动生成Getter/Setter方法，示例如下：

3. @NonNull：该注解用在属性或构造器上，Lombok会生成一个非空的声明，可用于校验参数，能帮助避免空指针。

4. @Cleanup：该注解能帮助我们自动调用close()方法，很大的简化了代码。

   ```java
   import lombok.Cleanup;
   import java.io.*;
   
   public class CleanupExample {
       public static void main(String[] args) throws IOException {
           @Cleanup InputStream in = new FileInputStream(args[0]);
           @Cleanup OutputStream out = new FileOutputStream(args[1]);
           byte[] b = new byte[10000];
           while (true) {
               int r = in.read(b);
               if (r == -1) break;
               out.write(b, 0, r);
           }
       }
   }
   ```

5. @EqualsAndHashCode：默认情况下，会使用所有非静态（non-static）和非瞬态（non-transient）属性来生成equals和hasCode，也能通过exclude注解来排除一些属性。

6. @ToString：类使用@ToString注解，Lombok会生成一个toString()方法，默认情况下，会输出类名、所有属性（会按照属性定义顺序），用逗号来分割。通过将`includeFieldNames`参数设为true，就能明确的输出toString()属性。

7. @NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor：无参构造器、部分参数构造器、全参构造器。Lombok没法实现多种参数构造器的重载。

Lombok工作原理：

1. 运行时解析：运行时能够解析的注解，必须将@Retention设置为RUNTIME，这样就可以通过反射拿到该注解。java.lang,reflect反射包中提供了一个接口AnnotatedElement，该接口定义了获取注解信息的几个方法，Class、Constructor、Field、Method、Package等都实现了该接口，对反射熟悉的朋友应该都会很熟悉这种解析方式。
2. 编译时解析：Lombok本质上就是一个实现了“[JSR 269 API](https://www.jcp.org/en/jsr/detail?id=269)”的程序。在使用javac的过程中，它产生作用的具体流程如下：
   1. javac对源代码进行分析，生成了一棵抽象语法树（AST）
   2. 运行过程中调用实现了“JSR 269 API”的Lombok程序
   3. 此时Lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树（AST），然后修改该语法树（AST），增加getter和setter方法定义的相应树节点
   4. javac使用修改后的抽象语法树（AST）生成字节码文件，即给class增加新的节点（代码块）

参考：[https://www.cnblogs.com/heyonggang/p/8638374.html](https://www.cnblogs.com/heyonggang/p/8638374.html)