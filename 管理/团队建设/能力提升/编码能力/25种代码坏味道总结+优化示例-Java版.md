# 25种代码坏味道总结+优化示例



### 前言

什么样的代码是好代码呢？好的代码应该命名规范、可读性强、扩展性强、健壮性......而不好的代码又有哪些典型特征呢？这25种代码坏味道大家要注意啦

### 1. Duplicated Code （重复代码）

重复代码就是**不同地点，有着相同的程序结构**。一般是因为需求迭代比较快，开发小伙伴担心影响已有功能，就复制粘贴造成的。重复代码**很难维护**的，如果你要修改其中一段的代码逻辑，就需要修改多次，很可能出现遗漏的情况。

如何优化重复代码呢？分**三种**情况讨论：

1. 同一个类的两个函数含有相同的表达式

```
class A {
    public void method1() {
        doSomething1
        doSomething2
        doSomething3
    }
    public void method2() {
        doSomething1
        doSomething2
        doSomething4
    }
}
```

优化手段：可以使用**Extract Method(提取公共函数)** 抽出重复的代码逻辑，组成一个公用的方法。

```
class A {
    public void method1() {
        commonMethod();
        doSomething3
    }
    public void method2() {
        commonMethod();
        doSomething4
    }

    public void commonMethod(){
       doSomething1
       doSomething2
    }
}
```

2. 两个互为兄弟的子类内含相同的表达式

```
class A extend C {
    public void method1() {
        doSomething1
        doSomething2
        doSomething3
    }
}

class B extend C {
    public void method1() {
        doSomething1
        doSomething2
        doSomething4
    }
}
```

优化手段：对两个类都使用**Extract Method(提取公共函数)**，然后把**抽取出来的函数放到父类**中。

```
class C {
    public void commonMethod(){
     doSomething1
     doSomething2
   }
}
class A extend C {
    public void method1() {
        commonMethod();
        doSomething3
    }
}

class B extend C {
    public void method1() {
        commonMethod();
        doSomething4
    }
}
```

3. 两个毫不相关的类出现重复代码

如果是两个毫不相关的类出现重复代码，可以使用**Extract Class**将重复代码提炼到一个类中。这个新类可以是一个普通类，也可以是一个工具类，看具体业务怎么划分吧。

### 2 .Long Method (长函数)

长函数是指一个函数方法几百行甚至上千行，可读性大大降低，不便于理解。**反例如下：**

```
public class Test {
    private String name;
    private Vector<Order> orders = new Vector<Order>();

    public void printOwing() {
        //print banner
        System.out.println("****************");
        System.out.println("*****customer Owes *****");
        System.out.println("****************");

        //calculate totalAmount
        Enumeration env = orders.elements();
        double totalAmount = 0.0;
        while (env.hasMoreElements()) {
            Order order = (Order) env.nextElement();
            totalAmount += order.getAmout();
        }

        //print details
        System.out.println("name:" + name);
        System.out.println("amount:" + totalAmount);
        ......
    }
}
```

可以使用`Extract Method`，抽取功能单一的代码段，组成命名清晰的小函数，去解决长函数问题，**正例如下**:

```
public class Test {
    private String name;
    private Vector<Order> orders = new Vector<Order>();

    public void printOwing() {

        //print banner
        printBanner();
        //calculate totalAmount
        double totalAmount = getTotalAmount();
        //print details
        printDetail(totalAmount);
    }

    void printBanner(){
        System.out.println("****************");
        System.out.println("*****customer Owes *****");
        System.out.println("****************");
    }

    double getTotalAmount(){
        Enumeration env = orders.elements();
        double totalAmount = 0.0;
        while (env.hasMoreElements()) {
            Order order = (Order) env.nextElement();
            totalAmount += order.getAmout();
        }
        return totalAmount;
    }

    void printDetail(double totalAmount){
        System.out.println("name:" + name);
        System.out.println("amount:" + totalAmount);
    }

}
```

### 3. Large Class (过大的类)

一个类做太多事情，维护了太多功能，可读性变差，性能也会下降。举个例子，订单相关的功能你放到一个类A里面，商品库存相关的也放在类A里面，积分相关的还放在类A里面..**.反例**如下：

```
Class A{
  public void printOrder(){
   System.out.println("订单");
  }

  public void printGoods(){
   System.out.println("商品");
  }

  public void printPoints(){
   System.out.println("积分");
  }
}
```

试想一下，乱七八糟的代码块都往一个类里面塞，还谈啥可读性。应该按单一职责，使用`Extract Class`把代码划分开，正例如下：

```
Class Order{
  public void printOrder(){
   System.out.println("订单");
  }
}

Class Goods{
   public void printGoods(){
   System.out.println("商品");
  }
}

Class Points{   
  public void printPoints(){
   System.out.println("积分");
  }
 }
}
```

### 4. Long Parameter List (过长参数列)

方法参数数量过多的话，可读性很差。如果有多个重载方法，参数很多的话，有时候你都不知道调哪个呢。并且，如果参数很多，做新老接口兼容处理也比较麻烦。

```
public void getUserInfo（String name,String age,String sex,String mobile){
  // do something ...
}
```

如何解决过长参数列问题呢？将参数封装成结构或者类，比如我们将参数封装成一个DTO类，如下：

```
public void getUserInfo（UserInfoParamDTO userInfoParamDTO){
  // do something ...
}

class UserInfoParamDTO{
  private String name;
  private String age; 
  private String sex;
  private String mobile;
}
```

### 5. Divergent Change （发散式变化）

对程序进行维护时, **如果添加修改组件, 要同时修改一个类中的多个方法**, 那么这就是 Divergent Change。举个汽车的例子，某个汽车厂商生产三种品牌的汽车：BMW、Benz和LaoSiLaiSi，每种品牌又可以选择燃油、纯电和混合动力。**反例如下**：

```
/**
 *  公众号：捡田螺的小男孩
 */
public class Car {

    private String name;

    void start(Engine engine) {
        if ("HybridEngine".equals(engine.getName())) {
            System.out.println("Start Hybrid Engine...");
        } else if ("GasolineEngine".equals(engine.getName())) {
            System.out.println("Start Gasoline Engine...");
        } else if ("ElectricEngine".equals(engine.getName())) {
            System.out.println("Start Electric Engine");
        }
    }

    void drive(Engine engine,Car car) {
        this.start(engine);
        System.out.println("Drive " + getBrand(car) + " car...");
    }

    String getBrand(Car car) {
        if ("Baoma".equals(car.getName())) {
            return "BMW";
        } else if ("BenChi".equals(car.getName())) {
            return "Benz";
        } else if ("LaoSiLaiSi".equals(car.getName())) {
            return "LaoSiLaiSi";
        }
        return null;
    }
 }
```

如果新增一种品牌新能源电车，然后它的启动引擎是核动力呢，那么就需要修改Car类的`start`和`getBrand`方法啦，这就是代码坏味道：**Divergent Change （发散式变化）**。

如何优化呢？一句话总结：**拆分类，将总是一起变化的东西放到一块**。

> - 运用提炼类(Extract Class) 拆分类的行为。
> - 如果不同的类有相同的行为，提炼超类(Extract Superclass) 和 提炼子类(Extract Subclass)。

**正例如下：**

因为Engine是独立变化的，所以提取一个Engine接口，如果新加一个启动引擎，多一个实现类即可。如下：

```
//IEngine
public interface IEngine {
    void start();
}

public class HybridEngineImpl implements IEngine { 
    @Override
    public void start() {
        System.out.println("Start Hybrid Engine...");
    }
}
```

因为`drive`方法依赖于`Car，IEngine，getBand`方法;`getBand`方法是变化的，也跟Car是有关联的，所以可以搞个抽象Car的类，每个品牌汽车继承于它即可，如下

```
public abstract class AbstractCar {

    protected IEngine engine;

    public AbstractCar(IEngine engine) {
        this.engine = engine;
    }

    public abstract void drive();
}

//奔驰汽车
public class BenzCar extends AbstractCar {

    public BenzCar(IEngine engine) {
        super(engine);
    }

    @Override
    public void drive() {
      this.engine.start();
      System.out.println("Drive " + getBrand() + " car...");
    }

    private String getBrand() {
        return "Benz";
    }
}

//宝马汽车
public class BaoMaCar extends AbstractCar {

    public BaoMaCar(IEngine engine) {
        super(engine);
    }

    @Override
    public void drive() {
        this.engine.start();
        System.out.println("Drive " + getBrand() + " car...");
    }

    private String getBrand() {
        return "BMW";
    }
}
```

细心的小伙伴，可以发现不同子类BaoMaCar和BenzCar的`drive`方法，还是有相同代码，所以我们可以再扩展一个抽象子类，把`drive`方法推进去，如下:

```
public abstract class AbstractRefinedCar extends AbstractCar {

    public AbstractRefinedCar(IEngine engine) {
        super(engine);
    }

    @Override
    public void drive() {
        this.engine.start();
        System.out.println("Drive " + getBrand() + " car...");
    }

    abstract String getBrand();
}

//宝马
public class BaoMaRefinedCar extends AbstractRefinedCar {

    public BaoMaRefinedCar(IEngine engine) {
        super(engine);
    }

    @Override
    String getBrand() {
        return  "BMW";
    }
}
```

如果再添加一个新品牌，搞个子类，继承`AbstractRefinedCar`即可，如果新增一种启动引擎，也是搞个类实现`IEngine`接口即可

### 6. Shotgun Surgery（散弹式修改）

当你实现某个小功能时，你需要在很多不同的类做出小修改。这就是**Shotgun Surgery（散弹式修改）**。它跟**发散式变化(Divergent Change)** 的区别就是，它指的是同时对多个类进行单一的修改，发散式变化指在一个类中修改多处。**反例如下**：

```
public class DbAUtils {
    @Value("${db.mysql.url}")
    private String mysqlDbUrl;
    ...
}

public class DbBUtils {
    @Value("${db.mysql.url}")
    private String mysqlDbUrl;
    ...
}
```

多个类使用了`db.mysql.url`这个变量，如果将来需要切换`mysql`到别的数据库，如`Oracle`，那就需要修改多个类的这个变量！

如何优化呢？将各个修改点，集中到一起，抽象成一个新类。

> 可以使用 Move Method （搬移函数）和 Move Field （搬移字段）把所有需要修改的代码放进同一个类，如果没有合适的类，就去new一个。

**正例如下：**

```
public class DbUtils {
    @Value("${db.mysql.url}")
    private String mysqlDbUrl;
    ...
}
```

### 7. Feature Envy (依恋情节)

某个函数为了计算某个值，从另一个对象那里调用几乎半打的取值函数。通俗点讲，就是一个函数使用了大量其他类的成员，有人称之为**红杏出墙的函数**。反例如下：

```
public class User{
    private Phone phone;
     public User(Phone phone){
        this.phone = phone;
    }
    public void getFullPhoneNumber(Phone phone){
        System.out.println("areaCode:" + phone.getAreaCode());
        System.out.println("prefix：" + phone.getPrefix());
        System.out.println("number：" + phone.getNumber());
    }
}
```

如何解决呢？在这种情况下，你可以考虑将这个方法移动到它使用的那个类中。例如，要将 `getFullPhoneNumber()` 从 `User` 类移动到`Phone` 类中，因为它调用了`Phone` 类的很多方法。

### 8. Data Clumps（数据泥团）

数据项就像小孩子，喜欢成群结队地呆在一块。如果一些数据项总是一起出现的，并且一起出现更有意义的，就可以考虑，按数据的业务含义来封装成数据对象。反例如下：

```
public class User {

    private String firstName;
    private String lastName;

    private String province;
    private String city;
    private String area;
    private String street;
}
```

正例：

```
public class User {

    private UserName username;
    private Adress adress;
}

class UserName{
    private String firstName;
    private String lastName;
}
class Address{
    private String province;
    private String city;
    private String area;
    private String street;
}
```

### 9. Primitive Obsession （基本类型偏执)

多数编程环境都有两种数据类型，**结构类型和基本类型**。这里的基本类型，如果指Java语言的话，不仅仅包括那八大基本类型哈，也包括String等。如果是经常一起出现的基本类型，可以考虑把它们封装成对象。我个人觉得它有点像**Data Clumps（数据泥团）** 举个反例如下：

```
// 订单
public class Order {
    private String customName;
    private String address;
    private Integer orderId;
    private Integer price;
}
```

正例：

```
// 订单类
public class Order {
    private Custom custom;
    private Integer orderId;
    private Integer price;
}
// 把custom相关字段封装起来，在Order中引用Custom对象
public class Custom {
    private String name;
    private String address;
}
```

当然，这里不是所有的基本类型，都建议封装成对象，有关联或者一起出现的，才这么建议哈。

### 10. Switch Statements （Switch 语句）

这里的Switch语句，不仅包括`Switch`相关的语句，也包括多层`if...else`的语句哈。很多时候，switch语句的问题就在于重复，如果你为它添加一个新的case语句，就必须找到所有的switch语句并且修改它们。

示例代码如下：

```
    String medalType = "guest";
    if ("guest".equals(medalType)) {
        System.out.println("嘉宾勋章");
     } else if ("vip".equals(medalType)) {
        System.out.println("会员勋章");
    } else if ("guard".equals(medalType)) {
        System.out.println("守护勋章");
    }
    ...
```

这种场景可以考虑使用多态优化：

```
//勋章接口
public interface IMedalService {
    void showMedal();
}

//守护勋章策略实现类
public class GuardMedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("展示守护勋章");
    }
}
//嘉宾勋章策略实现类
public class GuestMedalServiceImpl implements IMedalService {
    @Override
    public void showMedal() {
        System.out.println("嘉宾勋章");
    }
}

//勋章服务工厂类
public class MedalServicesFactory {

    private static final Map<String, IMedalService> map = new HashMap<>();
    static {
        map.put("guard", new GuardMedalServiceImpl());
        map.put("vip", new VipMedalServiceImpl());
        map.put("guest", new GuestMedalServiceImpl());
    }
    public static IMedalService getMedalService(String medalType) {
        return map.get(medalType);
    }
}
```

当然，多态只是优化的一个方案，一个方向。如果只是单一函数有些简单选择示例，并不建议动不动就使用动态，因为显得有点杀鸡使用牛刀了。

### 11.Parallel Inheritance Hierarchies（ 平行继承体系）

平行继承体系 其实算是`Shotgun Surgery`的特殊情况啦。当你为A类的一个子类Ax，也必须为另一个类B相应的增加一个子类Bx。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acb067c4657e41508d675a5b5657ca04~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

**解决方法**：遇到这种情况，就要消除两个继承体系之间的引用，有一个类是可以去掉继承关系的。

### 12. Lazy Class (冗赘类)

把这些不再重要的类里面的逻辑，合并到相关类，删掉旧的。一个比较常见的场景就是，假设系统已经有日期工具类`DateUtils`，有些小伙伴在开发中，需要用到日期转化等，不管三七二十一，又自己实现一个新的日期工具类。

### 13. Speculative Generality(夸夸其谈未来性)

尽量避免过度设计的代码。例如：

- 只有一个if else，那就不需要班门弄斧使用多态;
- 如果某个抽象类没有什么太大的作用，就运用`Collapse Hierarchy`（折叠继承体系）```
- 如果函数的某些参数没用上，就移除。

### 14. Temporary Field(令人迷惑的临时字段)

某个实例变量仅为某种特定情况而定而设，这样的代码就让人不易理解，我们称之为 `Temporary Field(令人迷惑的临时字段)`。 反例如下:

```
public class PhoneAccount {

    private double excessMinutesCharge;
    private static final double RATE = 8.0;

    public double computeBill(int minutesUsed, int includedMinutes) {
        excessMinutesCharge = 0.0;
        int excessMinutes = minutesUsed - includedMinutes;
        if (excessMinutes >= 1) {
            excessMinutesCharge = excessMinutes * RATE;
        }
        return excessMinutesCharge;
    }

    public double chargeForExcessMinutes(int minutesUsed, int includedMinutes) {
        computeBill(minutesUsed, includedMinutes);
        return excessMinutesCharge;
    }
}
```

思考一下，临时字段`excessMinutesCharge`是否多余呢？

### 15. Message Chains (过度耦合的消息链)

当你看到用户向一个对象请求另一个对象，然后再向后者请求另一个对象，然后再请求另一个对象...这就是消息链。实际代码中，你看到的可能是一长串`getThis（）`或一长串临时变量。反例如下：

```
A.getB().getC().getD().getTianLuoBoy().getData();
```

A想要获取需要的数据时，必须要知道B，又必须知道C,又必须知道D...其实A需要知道得太多啦，回头想下**封装性**，嘻嘻。其实可以通过拆函数或者移动函数解决，比如由B作为代理，搞个函数直接返回A需要数据。

### 16. Middle Man （中间人）

对象的基本特征之一就是封装，即对外部世界隐藏其内部细节。封装往往伴随委托，过度运用委托就不好：某个类接口有一半的函数都委托给其他类。可以使用`Remove Middle Man`优化。反例如下：

```
A.B.getC(){
   return C.getC();
}
```

其实，A可以直接通过C去获取C，而不需要通过B去获取。

### 17. Inappropriate Intimacy（狎昵关系）

如果两个类过于亲密，过分狎昵，你中有我，我中有你，两个类彼此使用对方的私有的东西，就是一种坏代码味道。我们称之为`Inappropriate Intimacy（狎昵关系）`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/215f432ac70b4b96816760a4ffdcad33~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

建议尽量把有关联的方法或属性抽离出来，放到公共类，以减少关联。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c70ca4d6124a4353b49a611687866318~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 18. Alternative Classes with Different Interfaces （异曲同工的类）

A类的接口a，和B类的接口b，做的的是相同一件事，或者类似的事情。我们就把A和B叫做异曲同工的类。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/869b55a09d87431da953417f59faf4ba~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

可以通过**重命名，移动函数，或抽象子类**等方式优化

### 19. Incomplete Library Class (不完美的类库)

大多数对象只要够用就好，如果类库构造得不够好，我们不可能修改其中的类使它完成我们希望完成的工作。可以酱紫：**包一层函数或包成新的类**。

### 20. Data Class （纯数据类）

什么是Data Class? 它们拥有一些字段，以及用于访问(读写)这些字段的函数。这些类很简单，仅有公共成员变量，或简单操作的函数。

如何优化呢？将**相关操作封装进去，减少public成员变量**。比如：

- 如果拥有public字段-> `Encapsulate Field`
- 如果这些类内含容器类的字段，应该检查它们是不是得到了恰当地封装-> `Encapsulate Collection` 封装起来
- 对于不该被其他类修改的字段-> `Remove Setting Method` ->找出取值/设置函数被其他类运用的地点-> `Move Method` 把这些调用行为搬移到`Data Class`来。如果无法搬移整个函数，就运用 `Extract Method`产生一个可被搬移的函数->`Hide Method`把这些取值/设置函数隐藏起来。

### 21. Refused Bequest （被拒绝的馈赠）

子类应该继承父类的数据和函数。子类继承得到所有函数和数据，却只使用了几个，那就是**继承体系设计错误**，需要优化。

- 需要为这个子类新建一个兄弟类->`Push Down Method`和`Push Down Field`把所有用不到的函数下推给兄弟类，这样一来，超类就只持有所有子类共享的东西。所有超类都应该是抽象的。
- 如果子类复用了超类的实现，又不愿意支持超类的接口，可以不以为然。但是不能胡乱修改继承体系->`Replace Inheritance with Delegation`(用委派替换继承).

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4446ed692d4409ea3c27e52de492655~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

### 22. Comments (过多的注释)

这个点不是说代码不建议写注释哦，而是，建议大家**避免用注释解释代码，避免过多的注释**。这些都是常见注释的坏味道：

- 多余的解释
- 日志式注释
- 用注释解释变量等
- ...

如何优化呢？

- 方法函数、变量的**命名要规范、浅显易懂**、避免用注释解释代码。
- **关键、复杂的业务**，使用**清晰、简明**的注释

### 23. 神奇命名

方法函数、变量、类名、模块等，都需要简单明了，浅显易懂。避免靠自己主观意识瞎起名字。

反例：

```
boolean test = chenkParamResult(req);
```

正例：

```
boolean isParamPass = chenkParamResult(req);
```

### 24. 神奇魔法数

日常开发中，经常会遇到这种代码：

```
if(userType==1){
   //doSth1
}else If( userType ==2){
   //doSth2
}
...
```

代码中的这个`1和2`都表示什么意思呢？再比如`setStatus(1)`中的`1`又表示什么意思呢？看到类似坏代码，可以这两种方式优化：

- **新建个常量类**，把一些常量放进去，统一管理，并且写好注释；
- **建一个枚举类**，把相关的魔法数字放到一起管理。

### 25. 混乱的代码层次调用

我们代码一般会分`dao层`、`service层`和`controller层`。

- dao层主要做数据持久层的工作，与数据库打交道。
- service层主要负责业务逻辑处理。
- controller层负责具体的业务模块流程的控制。

所以一般就是`controller`调用`service`，`service`调`dao`。如果你在代码看到`controller`直接调用`dao`，那可以考虑是否优化啦。**反例如下**：

```
@RestController("user")
public class UserController {

    Autowired
    private UserDao userDao;

    @RequestMapping("/queryUserInfo")
    public String queryUserInfo(String userName) {
        return userDao.selectByUserName(userName);
    }
}
```

### 参考资料

- [软工实验：常见的代码坏味道以及重构举例](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_45763536%2Farticle%2Fdetails%2F106315969%3Futm_medium%3Ddistribute.pc_relevant.none-task-blog-2%257Edefault%257EBlogCommendFromBaidu%257Edefault-5.vipsorttest%26depth_1-utm_source%3Ddistribute.pc_relevant.none-task-blog-2%257Edefault%257EBlogCommendFromBaidu%257Edefault-5.vipsorttest "https://blog.csdn.net/weixin_45763536/article/details/106315969?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-5.vipsorttest&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-5.vipsorttest")
- [22种代码的坏味道，一句话概括](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fwindcao%2Farticle%2Fdetails%2F25773219%3Futm_medium%3Ddistribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-15.nonecase%26depth_1-utm_source%3Ddistribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-15.nonecas "https://blog.csdn.net/windcao/article/details/25773219?utm_medium=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-15.nonecase&depth_1-utm_source=distribute.pc_relevant_download.none-task-blog-2~default~BlogCommendFromBaidu~default-15.nonecas")
- [【重构】 代码的坏味道总结 Bad Smell](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fshulianghan%2Farticle%2Fdetails%2F20009689 "https://blog.csdn.net/shulianghan/article/details/20009689")
- [Code Smell](https://link.juejin.cn?target=https%3A%2F%2Fwww.dazhuanlan.com%2F2019%2F12%2F26%2F5e04029e800c9%2F "https://www.dazhuanlan.com/2019/12/26/5e04029e800c9/")
- 《重构改善既有代码的设计》
