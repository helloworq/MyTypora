

# Dubbo学习记录

## Dubbo的XML配置文件运行demo

<img src="E:\DistCode\TyporaLoad\pic\wps1.png" alt="Dubbo框架" style="zoom:100%;" />

需求：生产者提供负责提供一个UserService方法，功能是打印一段话.消费者则是调用这个方法并打印一段话出来.

代码结构：将类的接口全部抽取到额外一个模块里减少冗余代码，所以总共有三个模块，如图.

![](E:\DistCode\TyporaLoad\pic\wps2.jpg)

然后开始实现一个最简单的dubbo样例。接口只写一个提供者的接口放在mydubboapi模块里，如图。

![](E:\DistCode\TyporaLoad\wps3.png)

接下来就是服务提供者的实现了，实现最简单的服务提供者需要三步：

第一步：创建一个用于提供服务的类，本例中是实现了mydubboapi里的UserService接口。如图。

![](E:\DistCode\TyporaLoad\wps4.jpg)

第二步：配置提供者的信息provider.xml。从文章开头可以看到提供者需要将自己的服务暴露给注册中心，告诉它我会做啥。

![](E:\DistCode\TyporaLoad\wps5.jpg)

如图可以看到有五个信息，分别来解释一下。

第一个信息是提供给dubboadmin图形化控制台的，告诉它我这个服务者叫啥，他好在控制台上给我个名字。

第二个信息可以说是告诉provider类zookeeper去哪里然后去注册自己的信息，别迷路了。

第三个信息就是协议了，因为不只有dubbo可以做这些。

第四个信息是声明暴露了哪个接口以及这个暴露接口的实现类在哪里。

第五个信息就是声明我的实现类在哪里。

怎么样是不是很简洁？

既然工作类有了，然后配置信息有了接下来就是激动人心的启动环节。启动代码如图所示。

![](E:\DistCode\TyporaLoad\wps6.jpg)

是不是简洁的不行？直接一个spring标志性的容器加载类加载一下provider.xml配置文件，再一个start就启动了！

然后就是更加简单的消费者类。如首页图所示，消费者只需要知道注册中心在哪里，然后告诉注册中心自己需要什么就行了。如图。

![](E:\DistCode\TyporaLoad\wps7.jpg)

前两个都不需要解释，第三个说下，id值就是zookeeper上注册的服务提供类的id。就是在provider.xml上配置的信息，后一个就是告诉你这个类的接口。然后就是激动人心的消费者启动了！如图。

![](E:\DistCode\TyporaLoad\wps8.jpg)

和提供者一样两行代码就启动了，然后调用的时候就像项目里其他普通类直接调用。

## 尝试注解配置

应用采用全注解的形式，不管提供者和消费者只需要将需要扫描到的注解所在的包的位置在启动类上配置好就可以。然后使用alibaba的@Service注解将一个类标识为一个服务，消费者调用的时候就使用@Reference注解将需要的服务注入进来就行了。

消费者启动类代码

```java

package com.mydubbo.dubbo;

import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.PropertySource;

@EnableCaching
@EnableDubbo(scanBasePackages = "com.mydubbo.dubbo")
@PropertySource("classpath:/application.properties")
@SpringBootApplication
public class DubboconsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboconsumerApplication.class, args);
    }
}
```

提供者启动类代码

```java
package com.mydubbo.dubbo;

import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@EnableDubbo(scanBasePackages = "com.mydubbo.dubbo.Service")
@SpringBootApplication
public class DubboproviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboproviderApplication.class, args);
    }
}
```

提供者配置文件

```properties
#mysql配置信息
server.servlet.context-path=/
spring.datasource.url=jdbc:oracle:thin:@127.0.0.1/ORCL
spring.datasource.username=dgp_ars
spring.datasource.password=pass
spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
#spring data jpa配置信息
spring.jpa.database=oracle
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
#redis配置信息
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.jedis.pool.max-wait=30000
spring.redis.jedis.pool.max-active=100
spring.redis.lettuce.pool.max-idle=20
spring.redis.jedis.pool.min-idle=0
spring.redis.timeout=3000
#dubbo配置
dubbo.application.name=user-service-provider
#注册中心地址
dubbo.registry.address=127.0.0.1:2181
#注解扫描地址
dubbo.scan.base-packages=com.mydubbo.dubbo
#注册中心协议
dubbo.registry.protocol=zookeeper
#协议名
dubbo.protocol.name=dubbo
#协议通信地址
dubbo.protocol.port=20881
#监控器协议
dubbo.monitor.protocol=registry
#Mongodb配置
```

提供者服务接口

```java
package com.mydubbo.dubbo.Service;

public interface SayhelloService {
    /**
     * 尝试自己的Service接口
     * @param info
     * @return
     */
    String printMsg(String info);
}

```

服务实现类

```java
package com.mydubbo.dubbo.Service.impl;

import com.alibaba.dubbo.config.annotation.Service;
import com.mydubbo.dubbo.Service.SayhelloService;
import org.springframework.stereotype.Component;

@Service//将服务暴露出来，注册到注册中心供消费者调用
@Component
public class SayhelloServiceImpl implements SayhelloService {
    @Override
    public String printMsg(String info) {
        return "hello"+info;
    }
}
```

消费者配置文件

```properties
server.port=8081

#消费者
#配置应用名
dubbo.application.name=boot-order-service-consumer
#注册中心地址
dubbo.registry.address=zookeeper://127.0.0.1:2181
#监控器协议
dubbo.monitor.protocol=registry
dubbo.consumer.timeout=30000
#dubbo.consumer.async=false
#日志

```

消费者消费类接口

```java
package com.mydubbo.dubbo.Controller;

import com.alibaba.dubbo.config.annotation.Reference;
import com.mydubbo.dubbo.Service.SayhelloService;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import java.util.List;

@Controller
public class getTestInfo {
    @Reference//将远程服务注入进来
    SayhelloService sayhelloService;

    @ResponseBody
    @RequestMapping("/getInfo")
    public String getInfo(){
        //直接调用远程服务的方法
        return sayhelloService.printMsg("成功了吗？");
    }
}
```



# Springboot集成Redis

## String类型

```
Redis五大类型:字符串（String）、哈希/散列/字典（Hash）、列表（List）、集合（Set）、有序集合（sorted set）五种
Controller:@Resource RedisTemplate<String, String> redisTemplate;
总括:
redisTemplate.opsForValue();//操作字符串
redisTemplate.opsForHash();//操作hash
redisTemplate.opsForList();//操作list
redisTemplate.opsForSet();//操作set
redisTemplate.opsForZSet();//操作有序set

String:
1.redisTemplate.opsForValue().set(key,value)); 
2.redisTemplate.opsForValue().get(key)); 
3.redisTemplate.opsForValue().get(key, start, end);
4.redisTemplate.opsForValue().getAndSet(key, value);
5.redisTemplate.opsForValue().getBit(key, offset);//下方注释
6.redisTemplate.opsForValue().multiGet(keys);
7.redisTemplate.opsForValue().setBit(key, offset, value);//下方注释
8.redisTemplate.opsForValue().set(K key, V value, long timeout, TimeUnit unit);//TimeUnit是timeout的类型,如毫秒\秒\天等
9.redisTemplate.opsForValue().setIfAbsent(key, value);
10.redisTemplate.opsForValue().set(K key, V value, long offset);//博主此处未做java验证
11.redisTemplate.opsForValue().size(key));
12.redisTemplate.opsForValue().multiGet(Collection<K> keys);
13.redisTemplate.opsForValue().multiSetIfAbsent(Map<? extends K, ? extends V> m);
14.同8
15\16\17\18\19.redisTemplate.opsForValue().increment(K key, long delta);或.increment(K key, double delta);
20.redisTemplate.opsForValue().append(key, value);//在key键对应值的右面追加值value
可以看到并没有删除等方法,博主研究了一下可以这样:21.del key------21.redisTemplate.opsForValue().getOperations().delete(key);
```

表格：

| 编号 | 命令                                                         |                  描述说明                  |
| ---- | ------------------------------------------------------------ | :----------------------------------------: |
| 1    | [SET key value](http://www.yiibai.com/redis/strings_set.html) |           此命令设置指定键的值。           |
| 2    | [GET key](http://www.yiibai.com/redis/strings_get.html)      |              获取指定键的值。              |
| 3    | [GETRANGE key start end](http://www.yiibai.com/redis/strings_getrange.html) |     获取存储在键上的字符串的子字符串。     |
| 4    | [GETSET key value](http://www.yiibai.com/redis/strings_getset.html) |       设置键的字符串值并返回其旧值。       |
| 5    | [GETBIT key offset](http://www.yiibai.com/redis/strings_getbit.html) |  返回在键处存储的字符串值中偏移处的位值。  |
| 6    | [MGET key1 [key2..\]](http://www.yiibai.com/redis/strings_mget.html) |             获取所有给定键的值             |
| 7    | [SETBIT key offset value](http://www.yiibai.com/redis/strings_setbit.html) | 存储在键上的字符串值中设置或清除偏移处的位 |
| 8    | [SETEX key seconds value](http://www.yiibai.com/redis/strings_setex.html) |          使用键和到期时间来设置值          |
| 9    | [SETNX key value](http://www.yiibai.com/redis/strings_setnx.html) |         设置键的值，仅当键不存在时         |
| 10   | [SETRANGE key offset value](http://www.yiibai.com/redis/strings_setrange.html) |  在指定偏移处开始的键处覆盖字符串的一部分  |
| 11   | [STRLEN key](http://www.yiibai.com/redis/strings_strlen.html) |          获取存储在键中的值的长度          |
| 12   | [MSET key value [key value …\]](http://www.yiibai.com/redis/strings_mset.html) |          为多个键分别设置它们的值          |
| 13   | [MSETNX key value [key value …\]](http://www.yiibai.com/redis/strings_msetnx.html) |  为多个键分别设置它们的值，仅当键不存在时  |
| 14   | [PSETEX key milliseconds value](http://www.yiibai.com/redis/strings_psetex.html) |     设置键的值和到期时间(以毫秒为单位)     |
| 15   | [INCR key](http://www.yiibai.com/redis/strings_incr.html)    |            将键的整数值增加`1`             |
| 16   | [INCRBY key increment](http://www.yiibai.com/redis/strings_incrby.html) |        将键的整数值按给定的数值增加        |
| 17   | [INCRBYFLOAT key increment](http://www.yiibai.com/redis/strings_incrbyfloat.html) |        将键的浮点值按给定的数值增加        |
| 18   | [DECR key](http://www.yiibai.com/redis/strings_decr.html)    |             将键的整数值减`1`              |
| 19   | [DECRBY key decrement](http://www.yiibai.com/redis/strings_decrby.html) |          按给定数值减少键的整数值          |
| 20   | [APPEND key value](http://www.yiibai.com/redis/strings_append.html) |              将指定值附加到键              |

## Hash类型

```
Hash:
1.redisTemplate.opsForHash().delete(H key, Object... hashKeys);//...表示可以传入多个map的key，用，隔开。或用数组传值
2.redisTemplate.opsForHash().hasKey(key, hashKey)；
3.redisTemplate.opsForHash().get(key, hashKey)；
4.redisTemplate.opsForHash().entries(key);//返回map集合
5、6.redisTemplate.opsForHash().increment(H key, HK hashKey, long delta);//或increment(H key, HK hashKey, double delta);；
7.redisTemplate.opsForHash().keys(key)；//返回map的key集合Set
8.redisTemplate.opsForHash().size(key)；
9.redisTemplate.opsForHash().multiGet(H key, Collection<HK> hashKeys);
10.redisTemplate.opsForHash().putAll(H key, Map<? extends HK, ? extends HV> m)；
11.redisTemplate.opsForHash().put(key, hashKey, value);
12.redisTemplate.opsForHash().putIfAbsent(key, hashKey, value)；
13.redisTemplate.opsForHash().values(key);//返回map中的value集合List；
```

表格：

| 序号 | 命令                                                         | 说明                                   |
| ---- | ------------------------------------------------------------ | -------------------------------------- |
| 1    | [HDEL key field2 [field2\]](http://www.yiibai.com/redis/hashes_hdel.html) | 删除一个或多个哈希字段。               |
| 2    | [HEXISTS key field](http://www.yiibai.com/redis/hashes_hexists.html) | 判断是否存在散列字段。                 |
| 3    | [HGET key field](http://www.yiibai.com/redis/hashes_hget.html) | 获取存储在指定键的哈希字段的值。       |
| 4    | [HGETALL key](http://www.yiibai.com/redis/hashes_hgetall.html) | 获取存储在指定键的哈希中的所有字段和值 |
| 5    | [HINCRBY key field increment](http://www.yiibai.com/redis/hashes_hincrby.html) | 将哈希字段的整数值按给定数字增加       |
| 6    | [HINCRBYFLOAT key field increment](http://www.yiibai.com/redis/hashes_hincrbyfloat.html) | 将哈希字段的浮点值按给定数值增加       |
| 7    | [HKEYS key](http://www.yiibai.com/redis/hashes_hkeys.html)   | 获取哈希中的所有字段                   |
| 8    | [HLEN key](http://www.yiibai.com/redis/hashes_hlen.html)     | 获取散列中的字段数量                   |
| 9    | [HMGET key field1 [field2\]](http://www.yiibai.com/redis/hashes_hmget.html) | 获取所有给定哈希字段的值               |
| 10   | [HMSET key field1 value1 [field2 value2 \]](http://www.yiibai.com/redis/hashes_hmset.html) | 为多个哈希字段分别设置它们的值         |
| 11   | [HSET key field value](http://www.yiibai.com/redis/hashes_hset.html) | 设置散列字段的字符串值                 |
| 12   | [HSETNX key field value](http://www.yiibai.com/redis/hashes_hsetnx.html) | 仅当字段不存在时，才设置散列字段的值   |
| 13   | [HVALS key](http://www.yiibai.com/redis/hashes_hvals.html)   | 获取哈希中的所有值                     |

## list类型

```
redisTemplate.opsForList().leftPush(key, value);//从左向右存压栈
redisTemplate.opsForList().leftPop(key);//从左出栈
redisTemplate.opsForList().size(key);//队/栈长
redisTemplate.opsForList().range(key, start, end);//范围检索,返回List
redisTemplate.opsForList().remove(key, i, value);//移除key中值为value的i个,返回删除的个数；如果没有这个元素则返回0 
redisTemplate.opsForList().index(key, index);//检索
redisTemplate.opsForList().set(key, index, value);//赋值
redisTemplate.opsForList().trim(key, start, end);//裁剪,void,删除除了[start,end]以外的所有元素  
redisTemplate.opsForList().rightPopAndLeftPush(String sourceKey, String destinationKey);//将源key的队列的右边的一个值删除，然后塞入目标key的队列的左边，返回这个值
注意:要缓存的对象必须实现Serializable接口,因为 Spring 会将对象先序列化再存入 Redis,否则报异常nested exception is java.lang.IllegalArgumentException: DefaultSerializer requires a Serializable……//；；/
```

表格

| 序号 | 命令                                                         | 说明                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | [BLPOP key1 [key2 \] timeout](http://www.yiibai.com/redis/lists_blpop.html) | 删除并获取列表中的第一个元素，或阻塞，直到有一个元素可用     |
| 2    | [BRPOP key1 [key2 \] timeout](http://www.yiibai.com/redis/lists_brpop.html) | 删除并获取列表中的最后一个元素，或阻塞，直到有一个元素可用   |
| 3    | [BRPOPLPUSH source destination timeout](http://www.yiibai.com/redis/lists_brpoplpush.html) | 从列表中弹出值，将其推送到另一个列表并返回它; 或阻塞，直到一个可用 |
| 4    | [LINDEX key index](http://www.yiibai.com/redis/lists_lindex.html) | 通过其索引从列表获取元素                                     |
| 5    | [LINSERT key BEFORE/AFTER pivot value](http://www.yiibai.com/redis/lists_linsert.html) | 在列表中的另一个元素之前或之后插入元素                       |
| 6    | [LLEN key](http://www.yiibai.com/redis/lists_llen.html)      | 获取列表的长度                                               |
| 7    | [LPOP key](http://www.yiibai.com/redis/lists_lpop.html)      | 删除并获取列表中的第一个元素                                 |
| 8    | [LPUSH key value1 [value2\]](http://www.yiibai.com/redis/lists_lpush.html) | 将一个或多个值添加到列表                                     |
| 9    | [LPUSHX key value](http://www.yiibai.com/redis/lists_lpushx.html) | 仅当列表存在时，才向列表添加值                               |
| 10   | [LRANGE key start stop](http://www.yiibai.com/redis/lists_lrange.html) | 从列表中获取一系列元素                                       |
| 11   | [LREM key count value](http://www.yiibai.com/redis/lists_lrem.html) | 从列表中删除元素                                             |
| 12   | [LSET key index value](http://www.yiibai.com/redis/lists_lset.html) | 通过索引在列表中设置元素的值                                 |
| 13   | [LTRIM key start stop](http://www.yiibai.com/redis/lists_ltrim.html) | 修剪列表的指定范围                                           |
| 14   | [RPOP key](http://www.yiibai.com/redis/lists_rpop.html)      | 删除并获取列表中的最后一个元素                               |
| 15   | [RPOPLPUSH source destination](http://www.yiibai.com/redis/lists_rpoplpush.html) | 删除列表中的最后一个元素，将其附加到另一个列表并返回         |
| 16   | [RPUSH key value1 [value2\]](http://www.yiibai.com/redis/lists_rpush.html) | 将一个或多个值附加到列表                                     |
| 17   | [RPUSHX key value](http://www.yiibai.com/redis/lists_rpushx.html) | 仅当列表存在时才将值附加到列表                               |

表格：

## Set类型

| 序号 | 命令                                                         | 说明                             |
| ---- | ------------------------------------------------------------ | -------------------------------- |
| 1    | [SADD key member1 [member2\]](http://www.yiibai.com/redis/sets_sadd.html) | 将一个或多个成员添加到集合       |
| 2    | [SCARD key](http://www.yiibai.com/redis/sets_scard.html)     | 获取集合中的成员数               |
| 3    | [SDIFF key1 [key2\]](http://www.yiibai.com/redis/sets_sdiff.html) | 减去多个集合                     |
| 4    | [SDIFFSTORE destination key1 [key2\]](http://www.yiibai.com/redis/sets_sdiffstore.html) | 减去多个集并将结果集存储在键中   |
| 5    | [SINTER key1 [key2\]](http://www.yiibai.com/redis/sets_sinter.html) | 相交多个集合                     |
| 6    | [SINTERSTORE destination key1 [key2\]](http://www.yiibai.com/redis/sets_sinterstore.html) | 交叉多个集合并将结果集存储在键中 |
| 7    | [SISMEMBER key member](http://www.yiibai.com/redis/sets_sismember.html) | 判断确定给定值是否是集合的成员   |
| 8    | [SMOVE source destination member](http://www.yiibai.com/redis/sets_smove.html) | 将成员从一个集合移动到另一个集合 |
| 9    | [SPOP key](http://www.yiibai.com/redis/sets_spop.html)       | 从集合中删除并返回随机成员       |
| 10   | [SRANDMEMBER key [count\]](http://www.yiibai.com/redis/sets_srandmember.html) | 从集合中获取一个或多个随机成员   |
| 11   | [SREM key member1 [member2\]](http://www.yiibai.com/redis/sets_srem.html) | 从集合中删除一个或多个成员       |
| 12   | [SUNION key1 [key2\]](http://www.yiibai.com/redis/sets_sunion.html) | 添加多个集合                     |
| 13   | [SUNIONSTORE destination key1 [key2\]](http://www.yiibai.com/redis/sets_sunionstore.html) | 添加多个集并将结果集存储在键中   |
| 14   | [SSCAN key cursor [MATCH pattern\] [COUNT count]](http://www.yiibai.com/redis/sets_sscan.html) | 递增地迭代集合中的元素           |

## Springboot+Redis的典型用法

```java
package com.mydubbo.dubbo;

import com.alibaba.dubbo.common.logger.Logger;
import com.alibaba.dubbo.common.logger.LoggerFactory;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.mydubbo.dubbo.Service.dao.CityInfoRepositry;
import com.mydubbo.dubbo.Service.dao.CityWeatherRepositry;
import com.mydubbo.dubbo.Service.util.DownLoad_Configs;
import com.mydubbo.dubbo.Service.util.GetUrlMessage;
import com.mydubbo.dubbo.bean.CityInfo;
import com.mydubbo.dubbo.bean.CityWeather;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

@Slf4j
@SpringBootTest
class DubboproviderApplicationTests{
    @Resource
    RedisTemplate<String,Object> redisTemplate;
    @Test
    void contextLoads() throws InterruptedException {
        /**
         * redis字符串操作
         */
        //redisTemplate.opsForValue().set("dasd","dasdsa");
        //redisTemplate.opsForValue().set("dsad","d",1);
        //System.out.println(redisTemplate.opsForValue().get("dsad"));
        //ArrayList<String> list=new ArrayList<>();
        //list.add("wdad");list.add("dsadasd");list.add("dsadasd");list.add("dsadasd");list.add("dsadasd");
        //redisTemplate.opsForList().leftPush("list",list);

        for (int i = 0; i < 10; i++) {
            long start=System.currentTimeMillis();
            String targetHTML=GetUrlMessage.getMessage("https://www.zhihu.com/question/287084175/answer/454611495");
            //System.out.println(redisTemplate.opsForList().leftPop("list"));
            String patt="(https?|ftp|file)://[-A-Za-z0-9+&@#/%?=~_|!:,.;]+[-A-Za-z0-9+&@#/%=~_|]";
            Pattern pattern=Pattern.compile(patt);
            Matcher matcher=pattern.matcher(targetHTML);
            while (matcher.find()) {
                redisTemplate.opsForSet().add("targetUrl",matcher.group());
            }
            //System.out.println(redisTemplate.opsForSet().members("targetUrl"));
            System.out.println("redis操作时间："+(System.currentTimeMillis()-start));
            Thread.sleep(1000);
        }

        for (int i = 0; i < 10; i++) {
            long start=System.currentTimeMillis();
            String targetHTML=GetUrlMessage.getMessage("https://www.zhihu.com/question/287084175/answer/454611495");
            //System.out.println(redisTemplate.opsForList().leftPop("list"));
            String patt="(https?|ftp|file)://[-A-Za-z0-9+&@#/%?=~_|!:,.;]+[-A-Za-z0-9+&@#/%=~_|]";
            Pattern pattern=Pattern.compile(patt);
            Matcher matcher=pattern.matcher(targetHTML);
            HashSet<String> hashSet=new HashSet<>();
            while (matcher.find()) {
                //redisTemplate.opsForSet().add("targetUrl",matcher.group());
                hashSet.add(matcher.group());
            }
            //System.out.println(redisTemplate.opsForSet().members("targetUrl"));
            System.out.println("hashset操作时间："+(System.currentTimeMillis()-start));
            Thread.sleep(1000);
        }
    }
}
```

Tips: **在使用RedisTemplate时需要使用javax包的@Resource注解注入到模板上**

## Redis的消息发布与订阅

在消费者模块里增加配置类如下：

```java
package com.mydubbo.dubbo.Config;

import com.mydubbo.dubbo.Service.impl.RedisReceiverServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.listener.PatternTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;

@Configuration
public class RedisListenerConfig {
    /**
     * redis消息监听器容器
     * 可以添加多个监听不同话题的redis监听器，只需要把消息监听器和相应的消息订阅处理器绑定，该消息监听器
     * 通过反射技术调用消息订阅处理器的相关方法进行一些业务处理
     * @param connectionFactory
     * @param listenerAdapter
     * @return
     */
    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory,
                                            MessageListenerAdapter listenerAdapter
    ) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);

        //可以添加多个 messageListener
        container.addMessageListener(listenerAdapter, new PatternTopic("index"));

        return container;
    }


    /**
     * 消息监听器适配器，绑定消息处理器，利用反射技术调用消息处理器的业务方法
     * @param redisReceiverService
     * @return
     */
    @Bean
    MessageListenerAdapter listenerAdapter(RedisReceiverServiceImpl redisReceiverService) {
        System.out.println("消息适配器进来了");
        return new MessageListenerAdapter(redisReceiverService, "receiveMessage");
    }

    //使用默认的工厂初始化redis操作模板
    @Bean
    StringRedisTemplate template(RedisConnectionFactory connectionFactory) {
        return new StringRedisTemplate(connectionFactory);
    }
}

```

暂时不写。

## Springboot+Redis实现缓存

# Springboot集成Mongodb

# Git命令

![](E:\DistCode\TyporaLoad\git-cheatsheet_1.png)

# 仿微博页面

强烈推荐[菜鸟教程](https://www.runoob.com/css/css-tutorial.html)

首先定义一个全局div将所有的元素都包在里面

```html
<div class="bodycontent" style="width: 1024px;height: auto; margin: 0 auto;"/>
```

在css里面的配置

```css
style="width: 1024px;height: auto; margin: 0 auto;"
```

加入layui的导航栏

```html
<div id="nav">
	<ul class="layui-nav">
	  <li class="layui-nav-item"><a href="">最新活动</a></li>
	  <li class="layui-nav-item layui-this">
		<a href="javascript:;">产品</a>
	  </li>
	  <li class="layui-nav-item"><a href="">大数据</a></li>
	  <li class="layui-nav-item">
		<a href="javascript:;">解决方案</a>
	  </li>
	  <li class="layui-nav-item"><a href="">社区</a></li>
	</ul>
</div>
```

加入长图

```html
<div class="weiboheader">
	<img class="weiboheaderpic" src="https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=14356876,365809084&fm=26&gp=0.jpg" />
	<div class="headONlongpicPosition">
		<img class="headONlongpic" src="https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=1418315404,2308746069&fm=26&gp=0.jpg">
	</div>
	<div class="operaONlongpicPosition">
		<button class="layui-btn layui-btn-danger"><i class="layui-icon">&#xe624;</i> 关注</button>
		<button class="layui-btn layui-btn-normal"><i class="layui-icon">&#xe609;</i> 私信</button>
	</div>
</div>
```

css样式

```css
		/*主样式，将div块的位置与父元素的居中位置相关联*/
		.weiboheader{
			position: relative;
		}
		/*控制图片大小以及与导航栏的间距*/
		.weiboheaderpic{
			margin-top: 10px;
			width: 1024px;
			height: 300px;
		}
		/*通过绝对定位控制头像位置*/
		/*因为div里的父元素已经与主父元素位置相关联所以绝对定位也是相对的*/
		.headONlongpicPosition {
			position: absolute;
			top: 50px;
			left: 462px
		}
		/*控制头像的大小以及形状*/
		.headONlongpic{
			width: 100px;
			height: 100px;
			border-radius: 100%;
		}
		/*通过绝对定位控制关注和私信的位置*/
		.operaONlongpicPosition{
			position: absolute;
			top: 200px;
			left: 425px
        }
```

接下来就是主要内容区域，我将区域大致分为了两块，

Leftpart     

![](E:\DistCode\TyporaLoad\微信截图_20200723094854.png)

maincontent

![](E:\DistCode\TyporaLoad\微信截图_20200723094901.png)

```css
leftpart通过这个样式占据左边
.leftpart{
	float: left;
	display: block;
	width: 30%;
	height: 10240px;
	margin-right: 10px;
}

相应的maincontent样式为
.maincontent{
	float: left;
	display: block;
	background-color: white;
	padding: 20px;
	width: 65%;
	height: auto;
	margin-top: 10px;
}
```

