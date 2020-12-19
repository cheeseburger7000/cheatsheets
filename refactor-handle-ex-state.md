# [重构] 优雅地处理异常和状态分支

[TOC]

## 面向扩展开发

todo

## 一个函数只做一件事

避免通过向函数传递一个“类型值”来控制函数的行为，这种做法通常意味着函数的主体不止一个。正确做法是将函数拆分开来，一个函数80%是控制逻辑，剩下的20%是真正要做的事。

## 如何处理异常逻辑和不同状态

函数里的异常分支尽早返回。

如果使用 if-else 通常说明 if 和 else 中的代码干的是同一件事情。例如：

```java
// 获取当前路径的所有文件夹
Map direcories = getDirectories("./");
appendLeafField(directories);

private void appendLeafField(Map directories) {
  for (Map directory: directories) {
    if (directory.containKey("children")) {
      directory.put("leaf", false);
      appendLeafField(directory);
    }
    else {
      directory.put("leaf", true);
    }
  }
}
```

把注意力放到 `appendLeafField` 这个递归函数上，这个递归函数的任务是给文件夹添加一个 `leaf` 字段表示当前文件是否是叶子结点，若文件下面没有子文件夹，那么就是叶子结点，否则就不是叶子结点。

```java
private void appendLeafField(Map directories) {
  for (Map directory: directories) {
    if (directory.containKey("children")) {
      directory.put("leaf", false);
      appendLeafField(directory);
      return;
    }
    directory.put("leaf", true);
  }
}
```

第二段代码中，虽然效果和第一段代码的执行结果是一样的，但是提早返回的那个 if 和下面的代码实际上**做的是同一件事情**，在这种情况下，使用 if/else 代码可读性更强些。

写 if/else 语句时尽量保证主干代码是正常的业务流程，异常情况提早返回，同时需要避免嵌套过深。

## 重构手法

### 1.合并条件表达式

for 异常逻辑处理

Before

```java
if ("admin".equals(username)) {
  throw new RumtimeException("不可以使用admin作为用户名");
}

if ("ADMIN".equals(username)) {
  throw new RumtimeException("不可以使用ADMIN作为用户名");
}

// aDmin, AdMin... is legally.
```

After

```java
if ("admin".equals(username) && "ADMIN".equals(username)) {
  throw new RumtimeException("不可以使用admin或ADMIN作为用户名");
}

// ...
```

下面这种情况使用合并条件表达式就显得不妥当，因为这两个 if 干的不是同一件事，拆分开来写可读性更强，同时方便后续进行扩展。

```java
if (!vaildateContent(params.content)) {
  throw new RuntimeException("存在非法内容");
}

if (isDuplicateUsername(params.username)) {
  throw new RuntimeException("用户名重复")
}

// Call svc to save user info to db...
```

### 2.卫语句

for 异常逻辑处理

使用卫语句将函数的异常分支提早返回，避免嵌套过深，使函数真正要做的事情清晰明了。

### 3.抽取函数

for 不同状态处理

```java
User user = getUser();
if ("admin".equals(user.type())) {
  // 处理管理员
} 
else if ("operation".equals(user.type())) {
  // 处理运营
}
else {
  // 处理普通用户
}
```

After

```java
User user = getUser();
if ("admin".equals(user.getType())) {
  handleAdmin();
} 
else if ("operation".equals(user.type())) {
  handleOperation();
}
else {
  handleNormal();
}

private void handleAdmin() {
  ...
}

private void handleOperation() {
  ...
}

private void handleNormalUser() {
  ...
}
```

### 4.抽取抽象类

for 不同状态处理

```java
abstract void handleUser();

class AdminHelper extends UserHelper {
  void handleUser() {
    ...
  }
}

class OperationHelper extends UserHelper {
  void handleUser() {
    ...
  }
}

switch(user.getType()) {
  case "admin":
    new AdminHelper().handle();
    break;
  case "operation":
    new OperationHelper().handle();
    break;
  default:
    new NormalUserHelper().handle();
}
```







