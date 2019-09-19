---
title: Spring Cache整合Redis
copyright: true
date: 2019-09-19 21:55:45
categories:
- [SpringBoot学习笔记]
- [Redis学习笔记]
tags:
- Spring Boot
- Redis
---

# 前言

缓存现在已经成为了互联网必不可少的利器，合理的利用缓存不仅能大大的提升网站的访问速度，还能够降低数据库的访问压力。

在上一篇文章中，介绍了 [如何在 SpringBoot 中整合 Redis](https://shangguanhong.github.io/2019/09/10/Spring-Boot整合Redis/)，接下来继续在其基础上介绍上如何利用 Spring Cache 与 Redis 来做缓存。

<!--more-->

# SpringCache整合Redis

依赖与 yml 配置与上篇文章相同，这里就不再给出了。Spring Cache 整合 Redis 最主要的是要修改 RedisConfig.java 配置文件。

新的 RedisConfig.java 文件内容如下

```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.cache.RedisCacheWriter;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.*;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

// 启用缓存功能
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    /**
     * redisTemplate相关配置
     *
     * @param factory factory
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 配置连接工厂
        template.setConnectionFactory(factory);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer<Object> serializer = getSerializer();
        // 值采用json序列化
        template.setValueSerializer(serializer);
        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(stringRedisSerializer);
        // 设置hash key 和value序列化模式
        template.setHashKeySerializer(stringRedisSerializer);
        template.setHashValueSerializer(serializer);
        template.afterPropertiesSet();
        return template;
    }

    @Override
    @Bean
    public KeyGenerator keyGenerator() {
        //为给定的方法及其参数生成一个键
        //格式为：target:method:[parms]
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());//类名
            sb.append(":");
            sb.append(method.getName());//方法名
            sb.append(":[");
            for (Object param : params) {
                sb.append(param.toString());//参数
                sb.append("-");
            }
            sb.append("]");
            return sb.toString();
        };
    }

    /**
     * 对hash类型的数据操作
     */
    @Bean
    public HashOperations<String, String, Object> hashOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForHash();
    }

    /**
     * 对redis字符串类型数据操作
     */
    @Bean
    public ValueOperations<String, Object> valueOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForValue();
    }

    /**
     * 对链表类型的数据操作
     */
    @Bean
    public ListOperations<String, Object> listOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForList();
    }

    /**
     * 对无序集合类型的数据操作
     */
    @Bean
    public SetOperations<String, Object> setOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForSet();
    }

    /**
     * 对有序集合类型的数据操作
     */
    @Bean
    public ZSetOperations<String, Object> zSetOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForZSet();
    }

    @Bean
    public RedisCacheConfiguration redisCacheConfiguration() {
        RedisSerializationContext.SerializationPair pair = RedisSerializationContext.SerializationPair
                .fromSerializer(getSerializer());
        return RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(300))
                .serializeValuesWith(pair);
    }

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        //初始化一个RedisCacheWriter
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(connectionFactory);
        //初始化RedisCacheManager
        RedisCacheManager cacheManager = new RedisCacheManager(redisCacheWriter, redisCacheConfiguration());
        return cacheManager;
    }

    /**
     * jackson序列化Redis
     */
    private Jackson2JsonRedisSerializer<Object> getSerializer() {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        return jackson2JsonRedisSerializer;
    }
}
```

至此，Redis 的配置已经完成，接下来就可以使用注解的方式来使用 Redis 缓存。

对于缓存声明，spring 提供了一组注解:

- @CacheConfig:设置类级别上共享的一些常见缓存设置。

- @Cacheable :触发缓存写入。
- @CacheEvict: 触发缓存清除。
- @CachePut: 更新缓存(不会影响到方法的运行)。
- @Caching :重新组合要应用于方法的多个缓存操作。

更多详情请查看 https://www.cnblogs.com/wenjunwei/p/10779450.html，下面的示例中用到了这几个注解。

UserService.java

```java
public interface UserService {

    User findById(Integer id);

    User update(User user);

    void deleteById(Integer id);
}
```

UserServiceImpl.java

```java
@Service
@CacheConfig(cacheNames = "user")
public class UserServiceImpl implements UserService {

    @Override
    @Cacheable(key = "#p0")
    public User findById(Integer id) {
        System.out.println("从数据库中查询用户");
        User user = new User();
        user.setId(id);
        user.setUsername("username:" + id.toString());
        return user;
    }

    @Override
    @CachePut(key = "#user.getId()")
    public User update(User user) {
        System.out.println("修改了id为" + user.getId() + "的用户");
        return user;
    }

    @Override
    @CacheEvict(key = "#p0")
    public void deleteById(Integer id) {
        System.out.println("删除了id为" + id + "的用户");
    }
}
```

下面进行 Redis 缓存测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisTest {
    
    @Resource
    private UserService userService;

    /**
     * redis缓存查找
     */
    @Test
    public void test1() {
        User user = userService.findById(1);
        System.out.println(user);
        User user1 = userService.findById(1);
        System.out.println(user1);
    }

    /**
     * redis缓存更新
     */
    @Test
    public void test2() {
        User user = userService.findById(2);
        System.out.println(user);
        user.setUsername("sgh");
        userService.update(user);
        user = userService.findById(2);
        System.out.println(user);
    }

    /**
     * redis缓存删除
     */
    @Test
    public void test3() {
        User user = userService.findById(1);
        System.out.println(user);
        userService.deleteById(1);
    }
}
```

详细信息请看项目地址：https://github.com/ShangguanHong/SpringBootDemo/tree/master/Redis

