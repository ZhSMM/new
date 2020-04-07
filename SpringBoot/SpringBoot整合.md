# Spring Boot整合

Spring Boot是操作数据：Spring-data jpa jdbc mongodb redis等；

在Spring Boot2.x之后，原来使用的jedis被替换成了lettuce：

- jedis：采用直连操作，多个线程操作不安全，如果要避免不安全，需要使用jedis pool连接池，类似BIO模式；
- lettuce：采用netty，实例可以在多个线程之中进行共享，不存在线程不安全的情况，可以减少线程数量，更像NIO模式；

```java
@Bean
@ConditionalOnMissingBean(
    name = {"redisTemplate"}
)
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    RedisTemplate<Object, Object> template = new RedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

@Bean
@ConditionalOnMissingBean
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```