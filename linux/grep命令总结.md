# grep命令

### Grep语法格式

```
第一种格式

	grep [option][pattern][file1,file2…]

第二种格式

	command | grep[option][pattern]
```

### Grep参数

* 常用

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/grep命令参数.png)

* 不常用

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/grep命令参数2.png)

### Grep和EGrep

1. grep默认不支持扩展正则表达式，只支持基础正则表达式
2. 使用grep -E可以支持扩展正则表达式
3. 使用egrep可以支持扩展正则表达式，与grep -E等价

### Grep命令脑图

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/grep命令脑图.png)

### 举例

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/grep示例.png)
