# Spring Boot任务

## 异步任务

1. 在service方法上添加`@Async`注解；
2. 在Application上添加`@EnableAsync`开启异步任务；

## 邮件发送

导入maven依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

简单使用：

```java
// 简单邮件
@org.junit.Test
public void test(){
    final SimpleMailMessage mailMessage=new SimpleMailMessage();
    mailMessage.setSubject("测试");
    mailMessage.setText("这是测试内容");
    mailMessage.setTo("1911472163@qq.com");
    mailMessage.setFrom("1911472163@qq.com");
    mailMessage.setReplyTo("1911472163@qq.com");
    mailSender.send(mailMessage);
}

// 复杂邮件
@org.junit.Test
public void test1() throws MessagingException {
    // 一个复杂邮件
    MimeMessage message=mailSender.createMimeMessage();
    // 组装
    MimeMessageHelper helper=new MimeMessageHelper(message,true);
    helper.setSubject("第二次测试");
    helper.setText("<p>带附件的复杂邮件</p>",true);

    // 发送方和接受方
    helper.setTo("1911472163@qq.com");
    helper.setFrom("1911472163@qq.com");
    helper.setReplyTo("1911472163@qq.com");
    // 附件
    helper.addAttachment("1.txt",new File("1.txt"));
    System.out.println(new File("").getAbsolutePath());
    mailSender.send(message);
}
```

## 定时任务

```
// 两个主要接口
TaskScheduler 任务调度程序
TaskExecutor 任务执行者

// 开启定时功能 
@EnableScheduling 
// 什么时候执行
// cron表达式：秒 分 时 日 月 周几
//   cron="0 0/5 11,18 * * ？"：每天的11点和18点每隔5分钟执行一次
@Scheduled(cron="0 39 11 * * ？") 
```

