# 第10章

[TOC]

## 10.4

### 命令执行的查找路径

1. 相对、绝对路径
2. alias
3. bash 内置命令
4. 环境变量 $PATH

观察 echo 的执行顺序

```bash
alias echo='echo -n'
type -a echo
```

### bash 的登录和欢迎信息👏

/etc/issue 登录、切换用户

/etc/motd 仅登录