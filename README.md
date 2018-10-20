# DDD设计思路应用
思路源于
https://mp.weixin.qq.com/s/c_5QUFu778NM67gNSrzvqA

### DDD设计模式带来的mock单元测试
如果你有做过大项目，你就会知道，一个大项目会微服务化，拆分为许多小项目，（具体怎么拆分，现在比较流行的是领域驱动设计，这里不作仔细描述）。微服务化带来的好处是，每个项目都极其简单，可能只有5到6张表，业务也不太复杂，对于互联网公司的快速迭代有天然的优势。但是麻烦也随着而来，一个小项目并不能单独工作，它需要依赖于其它域的项目。实际上，其它域的项目的日常环境并不能保证稳定，在开发、联调阶段经常出现调用其它域的接口故障，严重影响到开发效率。所以会有mock单元测试出来。也因为DDD设计模式下，用什么数据库并不重要，数据库只是个持久化的工具，所以有了Mock单元测试，下面介绍下我对Mock单元测试的应用。

### 引入maven
```xml
<dependency>
	<groupId>org.mockito</groupId>
	<artifactId>mockito-all</artifactId>
	<version>1.10.19</version>
	<scope>test</scope>
</dependency>
```

### 对服务类的要求
正常来说，一般我们引用一些类，都会胜过spring注解@AutoWired
如：
```java
public class A {
  @Autowired
  private B b;
}

```
但是如果你要对mock友好的话，必须要通过构造函数方式注入
如：
```java 
public class A {
  private B b;
  
  @AutoWired
  public A(B b) {
    this.b = b;
  }
}
```
### 开始写单元测试
```java 
  @RunWith(MockitoJUnitRunner.class)
  public class ATest {
    
    @Mock
    private B b;
    
    private A a;
    
    @Before
    public void init() {
      // 因为A类用了构造函数的方式注入服务，所以我们去模拟的时候也会变得很方便，直接new就可以把mock的类注入到要测试的类
      a = new a(b);
    }
    
    @Test
    public void test_action1(){
      // 这里就模拟了类b的getBillOrder方法，那么我们开发的时候就不会担心别人的接口能不能用，
      // 如果这里的接口是要访问数据库的，那么我们可以直接返回数据，而不用直接访问数据库存，是不是感到测试突然变得好方便。
      when(b.getBillOrder(anyString())).thenReturn(null);
      //这里可以直接调用要测试类A的方法了
      log.info(JSON.toJSONString(a.test()));
    }
  }
```

