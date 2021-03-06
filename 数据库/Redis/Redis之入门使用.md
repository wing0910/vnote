# 1. Redis入门使用
- 环境是Spring Boot 2.1.6+win10+mysql+redis
## 1.1. 加载依赖
```
 <!-- redis依赖包 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

## 1.2. 配置application.properties
```
#redis配置
#Redis服务器地址
spring.redis.host=127.0.0.1
#Redis服务器连接端口
spring.redis.port=6379
#Redis数据库索引（默认为0）
spring.redis.database=1  
# Redis密码
spring.redis.password=
#连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=50
#连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=3000
#连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=20
#连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=2
#连接超时时间（毫秒）
spring.redis.timeout=5000
```
## 1.3. 编写redis配置类
```
/**
 * @author Ada
 * @ClassName :RedisConfig
 * @date 2020/2/23 14:19
 * @Description: Redis操作类
 */
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        RedisTemplate template = new RedisTemplate();
        template.setConnectionFactory(factory);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();

        return template;
    }
}

```

## 1.4. 使用
比如在service层（业务逻辑层使用）

- 先注入一个RedisTemplate<String, String>对象
```
 @Autowired
 private RedisTemplate<String, String> redisTemplate;
```
- 在想要修改的方法里面添加

```
   **/
    @Override
    public BlogDetail getBlogDetail(Long blogId) {
        //redis的key键
        String blogRedisKey = "WEB:BLOG:" + blogId;
        Blog blog = new Blog();

        //判断是否有缓存
        if (redisTemplate.hasKey(blogRedisKey)) {
            blog.setBlogId(blogId);
            blog.setBlogTitle((String) redisTemplate.opsForHash().get(blogRedisKey, "blogTitle"));
            try {
                //将String类型的日期转化为Date类型
                SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                blog.setCreateTime(format.parse((String) redisTemplate.opsForHash().get(blogRedisKey, "createTime")));
                blog.setUpdateTime(format.parse((String) redisTemplate.opsForHash().get(blogRedisKey, "updateTime")));
            } catch (ParseException e) {
                e.printStackTrace();
            }
        } else {
            blog = blogMapper.selectByPrimaryKey(blogId);
            Map<String, Object> param = new HashMap<String, Object>();
            param.put("blogId", blogId.toString());
          
            //将datetime数据类型转化为String类型
            Date createTime = blog.getCreateTime();
            Date updateTime = blog.getUpdateTime();
            DateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            param.put("createTime", format.format(createTime));
            param.put("updateTime", format.format(updateTime));
            //value值是用hash类型
            redisTemplate.opsForHash().putAll(blogRedisKey, param);
            //设置过期时间
            redisTemplate.expire(blogRedisKey, 60 * 60, TimeUnit.SECONDS);
        }

        //不为空且状态已发布
        BlogDetail blogDetail = getBlogDetail(blog);
        if (blogDetail != null) {
            return blogDetail;
        }
        return null;
    }

```

## 1.5. 结果
打开redis的客户端就能够看到对应的缓存和缓存过期时间
![](_v_images/20200227171320957_503.png =966x)
