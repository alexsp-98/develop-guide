# 对象&类

## 1. 禁止使用JSON转化对象
JSON提供把对象转化为JSON字符串、把JSON字符串转为对象的功能，于是被用来转化对象。

这种对象转化方式，虽然在功能上没有问题，但是在性能上却存在问题。

反例
```java
List<UserDO> userDOList = ...;
List<UserVO> userVOList = JSON.parseArray(JSON.toJSONString(userDOList), UserVO.class);
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
## 2. 尽量不使用反射赋值对象
用反射赋值对象，主要优点是节省了代码量，主要缺点却是性能有所下降。

反例　
```java
List<UserDO> userDOList = ...;
List<UserVO> userVOList = new ArrayList<>(userDOList.size());
for (UserDO userDO : userDOList) {
    UserVO userVO = new UserVO();
    BeanUtils.copyProperties(userDO, userVO);
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
## 3. 采用Lambda表达式替换内部匿名类
对于大多数刚接触JDK8的同学来说，都会认为Lambda表达式就是匿名内部类的语法糖。

实际上， Lambda表达式在大多数虚拟机中采用invokeDynamic指令实现，相对于匿名内部类在效率上会更高一些。

反例
```java
List<User> userList = ...;
Collections.sort(userList, new Comparator<User>() {
    @Override
    public int compare(User user1, User user2) {
        Long userId1 = user1.getId();
        Long userId2 = user2.getId();
        ...
        return userId1.compareTo(userId2);
    }
});
```
正例
```java
List<User> userList = ...;
Collections.sort(userList, (user1, user2) -> {
    Long userId1 = user1.getId();
    Long userId2 = user2.getId();
    ...
    return userId1.compareTo(userId2);
});
```
## 4. 尽量避免定义不必要的子类
多一个类就需要多一份类加载，所以尽量避免定义不必要的子类。

反例
```java
public static final Map<String, Class> CLASS_MAP =
    Collections.unmodifiableMap(new HashMap<String, Class>(16) {
    private static final long serialVersionUID = 1L;
    {
        put("VARCHAR", java.lang.String.class);
    }
});
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
## 5. 尽量指定类的final修饰符
为类指定final修饰符，可以让该类不可以被继承。

如果指定了一个类为final，则该类所有的方法都是final的，Java编译器会寻找机会内联所有的final方法。

内联对于提升Java运行效率作用重大，具体可参见Java运行期优化，能够使性能平均提高50%。

反例
```java
public class DateUtils {
    ...
}
```
正例
```java
public final class DateUtils {
    ...
}
```
## 6. JSON序列化,Long类型被转成Integer类型
序列化为Json串后，Json串是没有Long类型的，而且反序列化回来如果也是Object接收，数字小于Integer最大值的话，会转成Integer！

演示代码
```java
public class JsonTest {

    public static void main(String[] args) {

        Long idValue = 3000L;
        Map<String, Object> data = new HashMap<>(2);
        data.put("id", idValue);
        data.put("name", "test");
        String jsonString = JSON.toJSONString(data);

        // 反序列化时Long被转为了Integer
        Map map = JSON.parseObject(jsonString, Map.class);
        Object idObj = map.get("id");
        System.out.println("反序列化的类型是否为Integer：" + (idObj instanceof Integer));
    }
}
```
运行结果
```java
反序列化的类型是否为Integer：true
```