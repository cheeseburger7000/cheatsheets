# 少即是多

[TOC]

# 语法

## 1. 三目表达式

```java
// 💩
String userType;
if (isMember(phoneNumber)) {
  userType = "会员";
}
else {
  userType = "游客";
}

// 🎉
String userType = isMember(phoneNumber) ? "会员" : "游客";
```

⚠️ 注意：对于包装类型的数值，拆包、自动拆包时要注意NPE

## 2. for-each

```java

double[] values = ...;
for(int i = 0; i < values.length; i++) {
    double value = values[i];
    // 需要通过idx遍历、还得担心数组下标越界
}

List<Double> valueList = ...;
Iterator<Double> iterator = valueList.iterator();
while (iterator.hasNext()) {
    Double value = iterator.next();
    // 操作迭代器逻辑太繁琐
}

List<Double> valueList = ...;
for(Double value : valueList) {
    // 最精简、优雅的方式
}
```

## 3. 利用 try-with-resource 关闭资源

```java
// 💩
BufferedReader reader = null;
try {
    reader = new BufferedReader(
      new FileReader("cities.csv"));
    String line = reader.readLine();
    while (line != null) {
        // ...
    }
} 
catch (IOException e) {
    log.error("读取文件异常", e);
} 
finally {
    if (reader != null) {
        try {
            reader.close();
        } 
      	catch (IOException e) {
            log.error("关闭文件异常", e);
        }
    }
}

// 🎉
try (BufferedReader reader = new BufferedReader(new FileReader("test.txt"))) {
    String line = reader.readLine();
    while (line != null) {
        // ...
    }
} 
catch (IOException e) {
    log.error("读取文件异常", e);
}
```

## 4. 卫语句

利用卫语句，将异常提前返回，避免定义中间变量

## 5. static 关键词

静态字段、静态函数，调用时无需初始化类对象。

常用在工具类中。

- [ ] 静态函数避免状态？
- [ ] 什么样的函数才可以定义为静态函数？

## 6. lambda 表达式

```java
new Thread(new Runnable() {
    public void run() {
        // ...
    }
}).start();

new Thread(() -> {
  // ...
}).start();
```

- [ ] 理解 Java 如何实现 filter、map等这些函数的
- [ ] 理解 Java 8 特性的概念，例如方法引用、日期等

## 7. 方法引用

使用方法引用简化 Lambda 表达式，省略变量声明和函数调用

```java
// 💩
Arrays.sort(nameArray, (a, b) -> a.compareToIgnoreCase(b));

List<Long> userIdList = userList
  	.stream()
    .map(user -> user.getId())
    .collect(Collectors.toList());

// 🎉
Arrays.sort(nameArray, String::compareToIgnoreCase);
List<Long> userIdList = userList
    .stream()
    .map(UserDO::getId)
    .collect(Collectors.toList());
```

## 8. 静态导入

```java
// 💩
List<Double> areaList = radiusList
  .stream()
  .map(r -> Math.PI * Math.pow(r, 2))
  .collect(Collectors.toList());

// 🎉
import static java.lang.Math.PI;
import static java.lang.Math.pow;
import static java.util.stream.Collectors.toList;

List<Double> areaList = radiusList
  .stream()
  .map(r -> PI * pow(r, 2))
  .collect(toList());
```

## 9. 利用非检查异常

非检查异常继承了RuntimeException ，特点是代码不需要处理它们也能通过编译。

利用 Unchecked 异常，可以避免不必要的 try-catch 和 throws 异常处理。

在 Web 开发下，非检查异常 + 全局异常处理器这种最佳实践处理异常。

# 注解

## 1. Lombok

Lombok 提供了许多用于消除大量模版代码的注解

```java
@Getter
@ToString
@Date
```

## 2. Validation 注解

```java
@Getter
@Setter
@ToString
public class UserCreateVO {
    @NotBlank(message = "用户名称不能为空")
    private String name;
    @NotNull(message = "公司标识不能为空")
    private Long companyId;
    // ...
}

@Service
@Validated
public class UserService {
    public Long createUser(@Valid UserCreateVO create) {
        // ...
    }
}
```

## 3. Spring 的 @NotNull

标注参数或返回值非空。

函数体的实现者需要考虑函数返回值的边界里是不包含 `null` 的。因此只要实现函数和调用方遵循规范，就可以避免不必要的空值判断。

📄 规范：函数不返回 `null`

👀 其他：Java8 的 Optional 特性

```java
Long companyId = 1L;
List<UserVO> userList = queryCompanyUser(companyId);
for (UserVO user : userList) {
    // ...
}

public @NonNull List<UserVO> queryCompanyUser(
  @NonNull Long companyId) {
    List<UserDO> userList = userDAO.queryByCompanyId(companyId);
    return userList
  		.stream()
      .map(this::transUser)
      .collect(Collectors.toList());
}
```

## 4. 注解特性

精解注解声明

1. 利用注解属性的默认值
2. 简写 value 属性（只有value属性时）
3. 使用组合注解

````java
// 💩
@Lazy(true);
@Service(value = "userService")
@RequestMapping(path = "/getUser", method = RequestMethod.GET)

// 🎉
@Lazy
@Service("userService")
@GetMapping("/getUser")
````

# 泛型

todo

# 函数

## 1.考虑函数返回值的边界

对于永远不会返回null的API，没有必要对返回值进行判空

```java
// 💩
List<User> users = apiSvc.getUsers(); // Return empty list if not found any users.
if (!CollectionUtils.isEmpty(users)) {
  List<String> names = user
    .stream()
    .map(User::getName)
    .collect(Collectors.toList());
}

// 🎉
List<User> users = apiSvc.getUsers();
List<String> names = user
  .stream()
  .map(User::getName)
  .collect(Collectors.toList());
```

## 2.使用QueryTable模式替代双重for循环

```java
private List<Product> getProductsWithCategory() {
	List<Category> categories = getAllCategories();
	List<Product> products = getAllProducts();
	for (Product product : products) {
		for (Category category : categories) {
			if (product.getCategoryId().equals(category.getId())) {
				product.setCategory(category);
			}
		}	
	}	
	for (Product product : products) {
		if (product.getCategory() == null) {
			product.setCategory(Category.uncategorized());
		}
	}
	return products;
}

// More clean then Nested for, right?
private List<Product> getProductsWithCategory() {
	Map<String, String> queryTable = new HashMap<>();
	List<Category> categories = getAllCategories();
	for (Category category : categories) {
		queryTable.put(category.getId(), category);
	}

	List<Product> products = getAllProducts();
	for (Product product : products) {
		Category category = queryTable.getOrDefault(product.getCategoryId(), Category.uncategorized());
		product.setCategory(category);
	}
	return products;
}
```

# 工具方法

## 1. 避免空值判断

```java
// 💩
if (users != null && !users.isEmpty()) {
    // ...
}

// 🎉
if (!CollectionUtils.isEmpty(users)) {
  // ...
}

if (StringUtils.isEmpty(username)) {
  // username == null 或 username.length() == 0 
}

if (StringUtils.isBlank(username)) {
  // username is empty or whitespace
}
```

## 2. 避免条件判断

```java
// 💩
double result;
if (value <= MIN_LIMIT) {
    result = MIN_LIMIT;
} 
else {
    result = value;
}

// 🎉
double result = Math.max(MIN_LIMIT, value);
```

## 3. 简化赋值语句

```java
// 💩
public static final List<String> ANIMAL_LIST;
static {
    List<String> animalList = new ArrayList<>();
    animalList.add("dog");
    animalList.add("cat");
    animalList.add("tiger");
    ANIMAL_LIST = Collections.unmodifiableList(animalList);
}

// 🎉
// JDK
public static final List<String> ANIMAL_LIST = Arrays.asList("dog", "cat", "tiger");
// Guava
public static final List<String> ANIMAL_LIST = ImmutableList.of("dog", "cat", "tiger");
```

## 4. 简化数据拷贝

```java
// 💩
UserVO userVO = new UserVO();
userVO.setId(userDO.getId());
userVO.setName(userDO.getName());
// ...
userVO.setDescription(userDO.getDescription());
userVOList.add(userVO);

// 💩💩💩 
// 性能损失过大。浅拷贝用不着JSON这样重量级的武器
List<UserVO> userVOList = JSON.parseArray(JSON.toJSONString(userDOList), UserVO.class);

// 🎉
UserVO userVO = new UserVO();
BeanUtils.copyProperties(userDO, userVO);
userVOList.add(userVO);
```

## 5. 简化异常断言

```java
// 💩
if (Objects.isNull(userId)) {
  throw new IllegalArgumentException(
    "用户标识不能为空");
}

// 🎉
Assert.notNull(userId, "用户标识不能为空");
```

## 6. 简化测试用例

将测试用例数据使用JSON格式存储到文件，然后使用Jacksin解析成对象，可减少大量的赋值语句。

```java
// 💩
@Test
public void testCreateUser() {
    UserCreateVO userCreate = new UserCreateVO();
    userCreate.setName("Changyi");
    userCreate.setTitle("Developer");
    userCreate.setCompany("AMAP");
    // ...
    Long userId  = userService.createUser(OPERATOR, userCreate);
    Assert.assertNotNull(userId, "创建用户失败");
}

// 🎉
@Test
public void testCreateUser() {
    String jsonText = ResourceHelper.getResourceAsString(getClass(), "createUser.json");
    UserCreateVO userCreate = JSON.parseObject(jsonText, UserCreateVO.class);
    Long userId  = userService.createUser(OPERATOR, userCreate);
    Assert.assertNotNull(userId, "创建用户失败");
}
```

## 7. 简化算法实现

一些常规算法，已经有现成的工具方法时，我们就没有必要自己实现了。

```java
// 💩
int totalSize = valueList.size();
List<List<Integer>> partitionList = new ArrayList<>();
for (int i = 0; i < totalSize; i += PARTITION_SIZE) {
    partitionList.add(valueList.subList(i, Math.min(i + PARTITION_SIZE, totalSize)));
}

// 🎉
List<List<Integer>> partitionList = ListUtils.partition(valueList, PARTITION_SIZE);
```

todo 了解 `ListUtils.partition` Guava 和 Apache 的工具类

## 8. 封装工具方法

特殊的算法，在没有现成的工具方法时，就得自己实现。

```java
// 💩
// SQL 设置参数值的方法就比较难用，setLong 方法不能设置参数值为 null 
if (Objects.nonNull(user.getId())) {
  statement.setLong(1, user.getId());
} 
else {
	statement.setNull(1, Types.BIGINT);
}

// 🎉
/** SQL辅助类 */
public final class SqlHelper {
    /** 设置长整数值 */
    public static void setLong(PreparedStatement statement, int index, Long value) throws SQLException {
        if (Objects.nonNull(value)) {
            statement.setLong(index, value.longValue());
        } 
      	else {
            statement.setNull(index, Types.BIGINT);
        }
    }
}

 // 设置参数值
SqlHelper.setLong(statement, 1, user.getId());
```

# 数据结构

## 1. Map简化

对于映射关系的 if/else 或 switch/case ，可以使用Map进行简化

```java
// 💩
public static String getBiologyClass(String name) {
    switch (name) {
        case "dog" :
            return "animal";
        case "cat" :
            return "animal";
        case "lavender" :
            return "plant";
        default :
            return null;
    }
}

// 🎉
private static final Map<String, String> BIOLOGY_CLASS_MAP
    = ImmutableMap.<String, String>builder()
        .put("dog", "animal")
        .put("cat", "animal")
        .put("lavender", "plant")
        .build();

public static String getBiologyClass(String name) {
    return BIOLOGY_CLASS_MAP.get(name);
}
```

## 2. 简化多个返回值

Java 不像 Python 和 Go ，方法不支持多个返回值。

需要多返回值时，就必须定义数据类。

可以使用Apache 提供的容器类：

1. Pair 支持返回 2 个对象
2. Triple 支持返回 3 个对象

```java
public static Pair<Point, Double> getNearest(Point point, 
	Point[] points) {
    // 计算最近点和距离
    // ...
    // 返回最近点和距离
    return ImmutablePair.of(nearestPoint, nearestDistance);
}
```

## 3. ThreadLocal

线程专有对象，可在整个线程的生命周期中随时取用。

使用 ThreadLocal 存储线程上下文对象，可以避免不必要的参数传递。

ThreadLocal 存在内存泄露的风险，尽量在业务代码结束之前调用 remove 方法进行数据清除。

# Optional

## 1. 保证值存在

```java
// 💩
Integer thisValue;
if (Objects.nonNull(value)) {
    thisValue = value;
} 
else {
    thisValue = DEFAULT_VALUE;
}

// 🎉
Integer thisValue = Optional
  .ofNullable(value)
  .orElse(DEFAULT_VALUE);
```

## 2. 保证值合法

```java
// 💩
Integer thisValue;
if (Objects.nonNull(value) 
    && value.compareTo(MAX_VALUE) <= 0) { // 合法条件
    thisValue = value;
} 
else {
    thisValue = MAX_VALUE;
}

// 🎉
Integer thisValue = Optional
  	.ofNullable(value)
    .filter(tempValue -> tempValue.compareTo(MAX_VALUE) <= 0)
  	.orElse(MAX_VALUE);

```

## 3. 避免空判断

```java
// 💩
String zipcode = null;
if (Objects.nonNull(user)) {
    Address address = user.getAddress();
    if (Objects.nonNull(address)) {
        Country country = address.getCountry();
        if (Objects.nonNull(country)) {
            zipcode = country.getZipcode();
        }
    }
}

// 🎉
tring zipcode = Optional
  .ofNullable(user)
  .map(User::getAddress)
  .map(Address::getCountry)
  .map(Country::getZipcode)
  .orElse(null);
```

# Stream

## 1. 匹配集合数据

```java
// 💩
boolean isFound = false;
for (UserDO user : userList) {
    if (Objects.equals(user.getId(), userId)) {
        isFound = true;
        break;
    }
}

// 🎉
boolean isFound = userList
  .stream()
  .anyMatch(user -> Objects.equals(user.getId(), userId));

```

## 2. 过滤集合数据

```java
// 💩
List<UserDO> result = new ArrayList<>();
for (UserDO user : users) {
    if (Boolean.TRUE.equals(user.getIsSuper())) {
        result.add(user);
    }
}

// 🎉
List<UserDO> result = users
  .stream()
  .filter(Boolean.TRUE.equals(user.getIsSuper()))
  .collect(Collectors.toList());
```

## 3. 汇总集合数据

```java
// 💩
double total = 0.0D;
for (Account account : accountList) {
    total += account.getBalance();
}

// 🎉
double total = accountList
  .stream()
  .mapToDouble(Account::getBalance)
  .sum();
```

## 4. 转化集合数据

```java
// 💩
List<UserVO> userVOList = new ArrayList<>();
for (UserDO userDO : userDOList) {
    userVOList.add(transUser(userDO));
}

// 🎉
List<UserVO> userVOList = userDOList
  .stream()
  .map(this::transUser)
  .collect(Collectors.toList());
```

## 5. 分组集合数据

```java
// 💩
Map<Long, List<UserDO>> roleUserMap = new HashMap<>();
for (UserDO userDO : userDOList) {
    roleUserMap
      .computeIfAbsent(userDO.getRoleId(), key -> new ArrayList<>())
      .add(userDO);
}

// 🎉
Map<Long, List<UserDO>> roleUserMap = userDOList
  .stream()
  .collect(Collectors.groupingBy(UserDO::getRoleId));
```

## 6. 分组汇总集合

```java
// 💩
Map<Long, Double> roleTotalMap = new HashMap<>();
for (Account account : accountList) {
    Long roleId = account.getRoleId();
    Double total = Optional.ofNullable(roleTotalMap.get(roleId)).orElse(0.0D);
    roleTotalMap.put(roleId, total + account.getBalance());
}

// 🎉
roleTotalMap = accountList
  .stream()
  .collect(Collectors.groupingBy(Account::getRoleId,                             			Collectors.summingDouble(Account::getBalance)));
```

## 7. 生成范围集合

```java
// 💩
int[] array1 = new int[N];
for (int i = 0; i < N; i++) {
    array1[i] = i + 1;
}

int[] array2 = new int[N];
array2[0] = 1;
for (int i = 1; i < N; i++) {
    array2[i] = array2[i - 1] * 2;
}

// 🎉
int[] array1 = IntStream
  .rangeClosed(1, N)
  .toArray();
int[] array2 = IntStream
  .iterate(1, n -> n * 2)
  .limit(N)
  .toArray();
```

# 程序结构

## 1. 直接返回条件表达式

```java
// 💩
public boolean isSuper(Long userId) {
    UserDO user = userDAO.get(userId);
    if (Objects.nonNull(user) 
        && Boolean.TRUE.equals(user.getIsSuper())) {
        return true;
    }
    return false;
}

// 🎉

public boolean isSuper(Long userId) {
    UserDO user = userDAO.get(userId);
    return Objects.nonNull(user) 
      && Boolean.TRUE.equals(user.getIsSuper());
}
```

## 2. 最小化条件作用域

```java
// 💩
Result result = summaryService.reportWorkDaily(workDaily);
if (result.isSuccess()) {
    String message = "上报工作日报成功";
    dingtalkService.sendMessage(user.getPhone(), message);
} 
else {
    String message = "上报工作日报失败:" + result.getMessage();
    log.warn(message);
    dingtalkService.sendMessage(user.getPhone(), message);
}

// 🎉
String message;
Result result = summaryService.reportWorkDaily(workDaily);
if (result.isSuccess()) {
    message = "上报工作日报成功";
} 
else {
    message = "上报工作日报失败:" + result.getMessage();
    log.warn(message);
}
// 提取出公共代码
dingtalkService.sendMessage(user.getPhone(), message);
```

## 3. 利用非空对象

```java
// 💩
username.equals("admin");

// 🎉
"admin".equals(username);
```

# 设计模式

## 1. 模版方法

## 2. 建造者模式

## 3. 代理模式

```java
// 💩
@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
    /** 用户服务 */
    @Autowired
    private UserService userService;

    /** 查询用户 */
    @PostMapping("/queryUser")
    public Result<?> queryUser(@RequestBody @Valid UserQueryVO query) {
        try {
            PageDataVO<UserVO> pageData = userService.queryUser(query);
            return Result.success(pageData);
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            return Result.failure(e.getMessage());
        }
    }
}
```

全局异常处理器

```java
@RestController
@RequestMapping("/user")
public class UserController {
    /** 用户服务 */
    @Autowired
    private UserService userService;

    /** 查询用户 */
    @PostMapping("/queryUser")
    public Result<PageDataVO<UserVO>> queryUser(
      @RequestBody @Valid UserQueryVO query) {
        PageDataVO<UserVO> pageData = userService.queryUser(query);
        return Result.success(pageData);
    }
}

@Slf4j
@ControllerAdvice
public class GlobalControllerAdvice {
    /** 处理异常 */
    @ResponseBody
    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error(e.getMessage(), e);
        return Result.failure(e.getMessage());
    }
}
```

基于AOP的异常处理

```java
@Slf4j
@Aspect
public class WebExceptionAspect {
    /** 点切面 */
    @Pointcut("@annotation(
       org.springframework.web.bind.annotation.RequestMapping)")
    private void webPointcut() {}

    /** 处理异常 */
    @AfterThrowing(pointcut = "webPointcut()", throwing = "e")
    public void handleException(Exception e) {
        Result<Void> result = Result.failure(e.getMessage());
        writeContent(JSON.toJSONString(result));
    }
}
```



# 删除代码

## 1. 删除已经废弃的代码

1. 包、导入
2. 类、字段、方法、变量、注解
3. 注释、被注释的代码（不实用的方法可以使用@Deprecated）
4. Maven导入、Mybatis的SQL语句
5. 配置文件的属性配置字段等

## 2. 删除接口方法的 public 关键词

## 3. 删除枚举构造方法的 private 关键词

枚举的构造方法都是 private 的，可不用显式声明

```java
// 💩
public enum UserStatus {
    DISABLED(0, "禁用"),
    ENABLED(1, "启用");
  
  	private final Integer value;
    private final String desc;
    
  	private UserStatus(Integer value, String desc) {
        this.value = value;
        this.desc = desc;
    }
}

// 🎉
public enum UserStatus {
    DISABLED(0, "禁用"),
    ENABLED(1, "启用");
    
  	private final Integer value;
    private final String desc;
    
  	UserStatus(Integer value, String desc) {
        this.value = value;
        this.desc = desc;
    }
}
```

## 4. 删除 final 类的方法的 final 关键词

final 类无法被继承，因此其方法不会被覆盖，没有必要添加 final 修饰

## 5. 删除不必要的变量

```java
// 💩
public Boolean existsUser(Long userId) {
    Boolean exists = userDAO.exists(userId);
    return exists;
}

// 🎉
public Boolean existsUser(Long userId) {
    return userDAO.exists(userId);
}
```

# 其他

- [CR是一场苦涩但有意思的修行 - 阿里孤尽](https://mp.weixin.qq.com/s?__biz=MzU4NzU0MDIzOQ==&mid=2247489170&idx=1&sn=e47dcf2227517172ff97105e8a0543d0&chksm=fdeb24f2ca9cade4985b11abd05d4c8e2fdf2cf9b5a73dbe27d320a036d684563679e8d5c565&token=1498852714&lang=zh_CN&scene=21#wechat_redirect)
- [如何写出无法维护的代码 - CoolShell](https://coolshell.cn/articles/4758.html#%E6%B5%8B%E8%AF%95)
