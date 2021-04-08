# 方法

## 1. 与类成员变量无关的方法声明成静态方法

静态方法的好处就是不用生成类的实例就可以直接调用。 静态方法不再属于某个对象，而是属于它所在的类。

只需要通过其类名就可以访问，不需要再消耗资源去反复创建对象。 即便在类内部的私有方法，如果没有使用到类成员变量，也应该声明为静态方法。

反例

```java
public int getLastMonth(){
    return LocalDate.now().plusMonths(-1).getMonthValue();
}
```

正例

```java
public static int getLastMonth(){
    return LocalDate.now().plusMonths(-1).getMonthValue();
}
```

## 2. 尽量使用基本数据类型作为方法参数类型，避免不必要的装箱、拆箱和空指针判断

反例

```java
public static float multi(Float value1,Float value2){
    float float1=Objects.isNull(value1)?0.0f:value1;
    float float2=Objects.isNull(value2)?0.0f:value2;
    return float1*float2;
}
float result=multi(2.0f,1.0f);
```

正例

```java
public static float multi(float value1, float value2) {
    return value1 * value2;
}
float result = multi(2.0f, 1.0f);
```

## 3. 尽量使用基本数据类型作为方法返回值类型，避免不必要的装箱、拆箱和空指针判断

在JDK类库的方法中，很多方法返回值都采用了基本数据类型，首先是为了避免不必要的装箱和拆箱，其次是为了避免返回值的空指针判断。

比如：Collection.isEmpty()和Map.size()。

反例

```java
public static Boolean isValid(UserDO user){
    if(Objects.isNull(user)){
        return false;
    }
    return Boolean.TRUE.equals(user.getIsValid());
}

// 调用代码
UserDO user=...;
Boolean isValid=isValid(user);
if(Objects.nonNull(isValid)&&isValid.booleanValue()){
    ...
}
```

正例

```java
public static boolean isValid(UserDO user){
    if(Objects.isNull(user)){
        return false;
    }
    return Boolean.TRUE.equals(user.getIsValid());
}

// 调用代码
UserDO user=...;
if(isValid(user)){
    ...
}
```

## 4. 协议方法参数值非空，避免不必要的空指针判断

协议编程，可以@NonNull和@Nullable标注参数，这样就能强制调用方对空指针异常进行预防，让我们的程序更为健壮。

反例

```java
public static boolean isValid(UserDO user){
    if(Objects.isNull(user)){
        return false;
    }
    return Boolean.TRUE.equals(user.getIsValid());
}
```

正例

```java
public static boolean isValid(@NonNull UserDO user){
    return Boolean.TRUE.equals(user.getIsValid());
}
```

## 5. 协议方法返回值非空，避免不必要的空指针判断

协议编程，可以@NonNull和@Nullable标注参数，这样就能强制调用方对空指针异常进行预防，让我们的程序更为健壮。

反例

```java
// 定义接口
public interface OrderService {
    public List<OrderVO> queryUserOrder(Long userId);
}

    // 调用代码
    List<OrderVO> orderList = orderService.queryUserOrder(userId);
if(CollectionUtils.isNotEmpty(orderList)){
    for(OrderVO order:orderList){
        ...
    }
}
```

正例

```java
// 定义接口
public interface OrderService {
    @NonNull
    public List<OrderVO> queryUserOrder(Long userId);
}

// 调用代码
List<OrderVO> orderList = orderService.queryUserOrder(userId);
for(OrderVO order:orderList){
    ...
}
```

## 6. 被调用方法已支持判空处理，调用方法无需再进行判空处理

反例

```java
UserDO user=null;
if(StringUtils.isNotBlank(value)){
    user=JSON.parseObject(value,UserDO.class);
}
```

正例

```java
UserDO user=JSON.parseObject(value,UserDO.class);
```

## 7. 尽量避免不必要的函数封装

方法调用会引起入栈和出栈，导致消耗更多的CPU和内存，应当尽量避免不必要的函数封装。

当然，为了使代码更简洁、更清晰、更易维护，增加一定的方法调用所带来的性能损耗是值得的。

反例

```java
// 函数封装
public static boolean isVip(Boolean isVip){
    return Boolean.TRUE.equals(isVip);
}

// 使用代码
boolean isVip=isVip(user.getVip());
```

正例

```java
boolean isVip=Boolean.TRUE.equals(user.getVip());
```

## 8. 尽量指定方法的final修饰符

方法指定final修饰符，可以让方法不可以被重写，Java编译器会寻找机会内联所有的final方法。

内联对于提升Java运行效率作用重大，具体可参见Java运行期优化，能够使性能平均提高50%。

注意：所有的private方法会隐式地被指定final修饰符，所以无须再为其指定final修饰符。

注意：使用Spring的AOP特性时，需要对Bean进行动态代理，如果方法添加了final修饰，将不会被代理。

例如：

```java
public class JSON {
    ...
    public static final int writeJSONString() {
        ...
    }
}
```
## 9. 方法拒绝过长的参数列表
方法过长的参数，不仅看起来不优雅，并且接口升级时，可能还要考虑新老版本兼容。
建议通过一个DTO来维护参数

例如：
```java
public void updateUserInfo（UserInfoDTO userInfoDTO) {
  // do something ...
}

class UserInfoDTO {
  private String name;
  private int age; 
  private String sex;
  private String mobile;
}
```
## 10. 优化你的程序逻辑，比如前面已经查到的数据，在后面的方法也用到的话，是可以把往下传参的，减少方法调用/查表
反例
```java
public ResultDTO updateUserInfo(UserParamDTO userParamDTO){
    
    UserInfoDO userInfoDO = userInfoMapper.selectUserByUserId(userParamDTO.getUserId());
     if(Objects.isNull(userInfoDO)){
       return null;
     }
   
    updateUserExpand(userParamDTO.getUserId());
    ...
}

private boolean updateUserExpand（String userId）{
    //又查了一次
    UserInfoDO userInfoDO = userInfoMapper.selectUserByUserId(userId);
    //更新用户头像
    updateUserHeadImg(userInfoDO);
    ....
}
```
正例
```java
public ResultDTO updateUserInfo(UserParamDTO userParamDTO){
    
    UserInfoDO userInfoDO = userInfoMapper.selectUserByUserId(userParamDTO.getUserId());
     if(Objects.isNull(userInfoDO)){
       return null;
     }
   
    updateUserExpand(userParamDTO.getUserId());
    ...
}

private boolean updateUserExpand（UserInfoDO userInfoDO）{
    //更新用户头像
    updateUserHeadImg(userInfoDO);
    ....
}
```
## 11. 优化程序结构，尽量减少方法的重复调用
反例
```java
public static void dealOrderList(List<OrderInfoDTO> orderInfoList) {
    for (int i = 0; i < orderInfoList.size(); i++) {
        //重复调用orderInfoList.size()方法了
    }
}
```
正例
```java
public static void dealOrderList(List<OrderInfoDTO> orderInfoList) {
    for (int i = 0, int length = orderInfoList.size(); i < length; i++) {
        //减少调用orderInfoList.size()方法，只在length变量调了一次。
    }
}
```
## 12. 对于行数比较多的函数，建议划分小函数，增强可读性。
一个过于冗长的函数或者一段需要注释才能让人理解用途的代码，可以考虑把它切分成一个功能明确的函数单元，并定义清晰简短的函数名，这样会让代码变得更加优雅。

反例
```java
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
    }
}
```
正例
```java
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
## 13. 当反射遇到方法重载的坑
如果不通过反射，传入Integer.valueOf(10)，走的是Integer重载。
然而，反射不是根据入参类型确定方法重载的，而是以反射获取方法时传入的方法名称和参数类型来确定的

示例代码
```java
public class ReflectTest {
    
    private void number(int number) {
        System.out.println("int number = " + number);
    }

    private void number(Integer number) {
        System.out.println("Integer number = " + number);
    }

    public static void main(String[] args) throws Exception {
        ReflectTest reflectTest = new ReflectTest();
        reflectTest.number(10);
        reflectTest.number(Integer.valueOf(10));

        reflectTest.getClass().getDeclaredMethod("number", Integer.TYPE).invoke(reflectTest, 20);
        reflectTest.getClass().getDeclaredMethod("number", Integer.class).invoke(reflectTest, 20);

        reflectTest.getClass().getDeclaredMethod("number", Integer.TYPE).invoke(reflectTest, Integer.valueOf("30"));
        reflectTest.getClass().getDeclaredMethod("number", Integer.class).invoke(reflectTest, Integer.valueOf("30"));
    }
}
```
运行结果
```java
int number = 10
Integer number = 10
int number = 20
Integer number = 20
int number = 30
Integer number = 30
```
## 14. 方法入参尽量都检验
入参校验也是每个程序员必备的基本素养。

你的方法处理，「必须先校验参数」。比如入参是否允许为空，入参长度是否符合你的预期长度。这个尽量养成习惯吧，很多「低级bug」都是「不校验参数」导致的。

假如你的数据库字段设置为varchar(16),对方传了一个32位的字符串过来，你不校验参数，「插入数据库直接异常」了。