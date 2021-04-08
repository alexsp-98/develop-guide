# 常量&变量

## 1. 直接赋值常量值，禁止声明新对象

直接赋值常量值，只是创建了一个对象引用，而这个对象引用指向常量值。这样减少了创建对象成本

反例

```java
Long i=new Long(1L);
String s=new String("abc");
```

正例

```java
Long i=1L;
String s="abc";
```

## 2. 当成员变量值无需改变时，尽量定义为静态常量

因为如果定义为static，即类静态常量，在每个实例对象中，它只有一份副本。

如果是成员变量，每个实例对象中，都各有一份副本。显然，如果这个变量不会变的话，定义为静态常量更好一些。

反例

```java
public class Constant {
    private final int retryTimes = 3;
}
```

正例

```java
public class Constant {
    private static final int RETRY_TIMES = 3;
}
```

## 3. 尽量使用基本数据类型，避免自动装箱和拆箱

Java 中的基本数据类型short、int、long、float、double、char、boolean， 分别对应包装类Short、Integer、Long、Float、Double、Character、Boolean。

JVM支持基本类型与对应包装类的自动转换，被称为自动装箱和拆箱。 装箱和拆箱都是需要CPU和内存资源的，所以应尽量避免使用自动装箱和拆箱。

反例

```java
Integer sum = 0;
int[] numbers = {1, 3, 4, 5};
for(int number : numbers) {
    sum += number;//相当于sum = Integer.valueOf(sum.intValue() + number);
}
```

正例

```java
int sum = 0;
int[] numbers = {1, 3, 4, 5};
for(int number : numbers) {
    sum += number;
}
```

## 4. 如果变量的初值会被覆盖，就没有必要给变量赋初值

反例
```java
List<UserDO> userList = new ArrayList<>();
if (isByMap) {
    userList = userMapper.selectList();
} else {
    userList = userMapper.selectByMap();
}
```
正例
```java
List<UserDO> userList;
if (isByMap) {
    userList = userMapper.selectList();
} else {
    userList = userMapper.selectByMap();
}
```

## 5. 尽量使用函数内的基本类型临时变量

在函数内，基本类型的参数和临时变量都保存在栈（Stack）中，访问速度较快；

对象类型的参数和临时变量的引用都保存在栈（Stack）中，内容都保存在堆（Heap）中，访问速度较慢。

在类中，任何类型的成员变量都保存在堆（Heap）中，访问速度较慢。

反例
```java
public class Calculate {
    private int result = 0;
   
    public void addAll(@NonNull int[] numbers) {
        for(int number : numbers) {
            result += value;
        }
    }
    ...
}
```
正例
```java
public class Calculate {
    private int result = 0;
   
    public void addAll(@NonNull int[] numbers) {
        int sum = 0;
        for(int number : numbers) {
            sum += value;
        }
        result = sum;
    }
    ...
}
```

## 6. 尽量不要在循环体外定义变量

在老版JDK中，建议“尽量不要在循环体内定义变量”，但是在新版的JDK中已经做了优化。

通过对编译后的字节码分析，变量定义在循环体外和循环体内没有本质的区别，运行效率基本上是一样的。

反而，根据“ 局部变量作用域最小化 ”原则，变量定义在循环体内更科学更便于维护，避免了延长大对象生命周期导致延缓回收问题 。

[Java变量声明在循环体内还是循环体外，你用哪一个？](https://www.163.com/dy/article/FQGITL740511FQO9.html)

反例
```java
UserVO userVO;
List<UserDO> userDOList = ...;
List<UserVO> userVOList = new ArrayList<>(userDOList.size());
for (UserDO userDO : userDOList) {
    userVO = new UserVO();
    userVO.setId(userDO.getId());
    ...
    userVOList.add(userVO);
}
```
正例
```java
List<UserDO> userDOList = ...;
List<UserVO> userVOList = new ArrayList<>(userDOList.size());
for (UserDO userDO : userDOList) {
    UserVO userVO = new UserVO();
    userVO.setId(userDO.getId());
    ...
    userVOList.add(userVO);
}
```
## 7. 不可变的静态常量，尽量使用非线程安全类

不可变的静态常量，虽然需要支持多线程访问，也可以使用非线程安全类。

反例
```java
public static final Map<String, Class> CLASS_MAP;
static {
    Map<String, Class> classMap = new ConcurrentHashMap<>(16);
    classMap.put("VARCHAR", java.lang.String.class);
    ...
    CLASS_MAP = Collections.unmodifiableMap(classMap);
}
```
正例
```java
public static final Map<String, Class> CLASS_MAP;
static {
    Map<String, Class> classMap = new HashMap<>(16);
    classMap.put("VARCHAR", java.lang.String.class);
    ...
    CLASS_MAP = Collections.unmodifiableMap(classMap);
}
```

## 8. 不可变的成员变量，尽量使用非线程安全类

不可变的成员变量，虽然需要支持多线程访问，也可以使用非线程安全类。

反例
```java
@Service
public class StrategyFactory implements InitializingBean {
    @Autowired
    private List<Strategy> strategyList;
    private Map<String, Strategy> strategyMap;
    @Override
    public void afterPropertiesSet() {
        if (CollectionUtils.isNotEmpty(strategyList)) {
            int size = (int) Math.ceil(strategyList.size() * 4.0 / 3);
            Map<String, Strategy> map = new ConcurrentHashMap<>(size);
            for (Strategy strategy : strategyList) {
                map.put(strategy.getType(), strategy);
            }
            strategyMap = Collections.unmodifiableMap(map);
        }
    }
    ...
```
正例
```java
@Service
public class StrategyFactory implements InitializingBean {
    @Autowired
    private List<Strategy> strategyList;
    private Map<String, Strategy> strategyMap;
    @Override
    public void afterPropertiesSet() {
        if (CollectionUtils.isNotEmpty(strategyList)) {
            int size = (int) Math.ceil(strategyList.size() * 4.0 / 3);
            Map<String, Strategy> map = new HashMap<>(size);
            for (Strategy strategy : strategyList) {
                map.put(strategy.getType(), strategy);
            }
            strategyMap = Collections.unmodifiableMap(map);
        }
    }
    ...
}
```
## 9. 不要直接在代码中使用0,1等魔法值，应该要用enum枚举代替

反例
```java
if("0".equals(order.getOrderStatus)){
    //未支付，去支付
    toPay(order);
}else if("1".equals(order.getOrderStatus)){
    //已支付，获取订单详情
    getOrder(order);
}
```
正例
```java
if(PayStatusEnum.NOT_PAY.getCode.equals(order.getPayStatus)){
    //未支付，去支付
    toPay(order);
}else if(PayStatusEnum.PAID.getCode.equals(order.getPayStatus)){
    //已支付，获取订单详情
    getOrder(order);
}

public enum PayStatusEnum {

    NOT_PAY("0","未支付"),
    PAID("1","已支付");

    private String code; 
    private String desc;

    PayStatusEnum(String code, String desc) {
        this.code = code;
        this.desc = desc;
    }
}
```

## 10. static静态变量不要依赖spring实例化变量，可能会导致初始化出错

定义静态类做初始化时，切忌做强制依赖spring容器，因为类加载顺序不是确定的，会导致实际使用该类时因为没有初始化导致程序出错

反例
```java
private static UserService userService = SpringContextUtils.getBean(UserService.class);
```

正例
```java
private static UserService userService = null;

public static UserService getUserService(){
    if(userService == null){
        userService = SpringContextUtils.getBean(UserService.class);
    }
    return userService;
}
```

## 11. 金额数值计算要使用BigDecimal

计算机是以二进制存储数值的，浮点数也是。而0.1对于计算机来说无法精确表达，这就是为什么浮点数会导致精确度缺失的原因。

因此，涉及到金额计算，一般都是用BigDecimal类型

反例
```java
System.out.println(0.7 + 0.2);
System.out.println(0.7 - 0.2);
System.out.println(0.7 * 0.2);
System.out.println(0.7 / 0.2);

double number1 = 0.7;
double number2 = 0.2;
double result = number1 - number2;
if (result == 0.5){
    System.out.println("Test ok");
} else {
    System.out.println("Test fail, result:" + result);
}
```
运行结果
```java
0.8999999999999999
0.49999999999999994
0.13999999999999999
3.4999999999999996
Test fail, result:0.49999999999999994
```

正例
```java
System.out.println(new BigDecimal("0.7").add(new BigDecimal("0.2")));
```
运行结果
```java
0.9
```
另外需要注意的是，使用BigDecimal表示和计算浮点数，必须使用字符串的构造方法来初始化BigDecimal，其他方式也会缺失精度
```java
System.out.println(new BigDecimal(0.7).add(new BigDecimal(0.2)));
```
运行结果
```java
0.899999999999999966693309261245303787291049957275390625
```
同时要关注BigDecimal的几位小数点，它有八种舍入模式：

|        舍入模式          | 值  |                                       说明                                             |
| ---------------------- | --- | -------------------------------------------------------------------------------------- |
|ROUND_UP                |  0  | 进位处理，不管后面数字的大小                                                                |
|ROUND_DOWN              |  1  | 去掉多余的位数，不管后面数字的大小                                                           |
|ROUND_CEILING           |  2  | 如果是正数，相当于BigDecimal.ROUND_UP;如果是负数，相当于BigDecimal.ROUND_DOWN                |
|ROUND_FLOOR             |  3  | 如果是正数，相当于BigDecimal.ROUND_DOWN;如果是负数，相当于BigDecimal.ROUND_HALF_UP           |
|ROUND_HALF_UP           |  4  | 根据后面的数字四舍五入（大于等于5进位）                                                       |
|ROUND_HALF_DOWN         |  5  | 根据后面的数字四舍五入（大于5进位）                                                          |
|ROUND_HALF_EVEN         |  6  | 如果舍弃部分左边的数字为偶数，则作 ROUND_HALF_DOWN;如果舍弃部分左边的数字为奇数，则作 ROUND_HALF_UP |
|ROUND_UNNECESSARY       |  7  | Scale位数必须小于等于小数位数，否则会抛异常                                                   |

## 12. 尽量减少对变量的重复计算

尽量减少变量的重复计算，对方法的调用，即使是只有一个语句，也是有消耗的，比如创建栈帧。

如果list比较大时，多次调用list.size也是会有资源消耗的。

反例
```java
for (int i = 0; i < list.size(); i++){

}
```
正例
```java
for (int i = 0, int length = list.size(); i < length; i++){

}
```