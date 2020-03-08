# MySql连接出现时区问题

在连接MySQL数据库时，会出现时区错误，解决方法有两种。

## 修改MySQL时区

进入MySQL命令模式，执行以下命令：

```
// 就能解决连接问题
set global time_zone='+8:00';
flush privileges;
```

### 查看时区

输入`show variables like '%time_zone%';`

### 全局时区设置

`set global time_zone='+8:00';`

### 仅设置当前会话

即推出后设置会消失：`set time_zone='+8:00';`

### 立即生效

`flush privileges;`

### 时区

- UTC是协调世界时(Universal Time Coordinated)

- GMT(Greenwich Mean Time)是格林尼治平时

- 而CST却同时可以代表如下 4 个不同的时区： 
  - 美国中部时间：Central Standard Time (USA) UT-6:00
  - 澳大利亚中部时间：Central Standard Time (Australia) UT+9:30
  - 中国标准时间：China Standard Time UT+8:00
  - 古巴标准时间：Cuba Standard Time UT-4:00

## 连接时设置

即将url修改为：`jdbc:mysql://localhost:3306/test?serverTimezone=UTC`，通常还会避免乱码，因此通常还会加上`&useUnicode=true&characterEncoding=UTF-8`，即为：

`jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=UTF-8`