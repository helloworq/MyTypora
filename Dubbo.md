# Dubbo学习记录

## Dubbo的XML配置文件运行demo

<img src="C:\Users\12733\Desktop\wps1.png" alt="Dubbo框架" style="zoom:100%;" />

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

### 普通cache缓存

#### 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

#### 启动类加上注解

```java
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

#### @Cacheable

```java
@Cacheable(value = "emp" ,key = "targetClass + methodName +#p0")
public List<NewJob> queryAll(User uid) {
    return newJobDao.findAllByUid(uid);
}

/*
此处的User实体类一定要实现序列化public class User implements Serializable，否则会报java.io.NotSerializableException异常。
*/
```

#### @Cacheput

```java
/*
@CachePut注解的作用 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存，和 @Cacheable 不同的是，它每次都会触发真实方法的调用 。简单来说就是用户更新缓存数据。但需要注意的是该注解的value 和 key 必须与要更新的缓存相同，也就是与@Cacheable 相同。示例：
*/
@CachePut(value = "emp", key = "targetClass + #p0")
public NewJob updata(NewJob job) {
    //查询数据库更新缓存
    NewJob newJob = newJobDao.findAllById(job.getId());
    newJob.updata(job);
    return job;
}

@Cacheable(value = "emp", key = "targetClass +#p0")
public NewJob save(NewJob job) {
    newJobDao.save(job);
    return job;
}
```

#### @CacheEvict

```java
//@CachEvict的作用 主要针对方法配置，能够根据一定的条件对缓存进行清空 。
@Cacheable(value = "emp",key = "#p0.id")
public NewJob save(NewJob job) {
    newJobDao.save(job);
    return job;
}

//清除一条缓存，key为要清空的数据
@CacheEvict(value="emp",key="#id")
public void delect(int id) {
    newJobDao.deleteAllById(id);
}

//方法调用后清空所有缓存
@CacheEvict(value="accountCache",allEntries=true)
public void delectAll() {
    newJobDao.deleteAll();
}

//方法调用前清空所有缓存
@CacheEvict(value="accountCache",beforeInvocation=true)
public void delectAll() {
    newJobDao.deleteAll();
}
```

#### 组合Caching

```java
@Caching(
cacheable = {
        @Cacheable(value = "emp",key = "#p0"),
        ...
},
put = {
        @CachePut(value = "emp",key = "#p0"),
        ...
},evict = {
        @CacheEvict(value = "emp",key = "#p0"),
        ....
})
public User save(User user) {
    ....
}
```

### 使用Redis缓存

```java
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        return RedisCacheManager.create(factory);
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(factory);
        return redisTemplate;
    }
}
```



# Git命令

Git命令一览：

![](E:\DistCode\TyporaLoad\git-cheatsheet_1.png)

## Git命令详细

拥有一个版本库的方法有如下几个：

* cd进本地电脑的任意文件夹然后打开bash直接从远程仓库clone一个仓库下来

  ```
  git clone git@github.com:michaelliao/gitskills.git 将远程仓库克隆下来
  ```

* 本地也可以创建自己一个版本库，cd进任一目录，使用 git init 命令就可以直接将此目录及以下的文件加入版本库，然后想将本地仓库关联到远程仓库的话就需要先在远程仓库创建一个仓库，然后使用如下命令进行关联.

  ```
  git remote add origin git@github.com:helloworq/WeiBoSurface.git
  ```

  这里可以看到有账户信息，显然我们需要配置一下自己的信息告诉远程仓库我是谁，不然谁都可以关联仓库。

* 第1步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

  ```
  $ ssh-keygen -t rsa -C "youremail@example.com"
  ```

  你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。

  如果一切顺利的话，可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，`id_rsa`是私钥，不能泄露出去，`id_rsa.pub`是公钥，可以放心地告诉任何人。

  第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面：

  然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容：

  ![github-addkey-1](https://www.liaoxuefeng.com/files/attachments/919021379029408/0)

  点“Add Key”，你就应该看到已经添加的Key：

  ![github-addkey-2](https://www.liaoxuefeng.com/files/attachments/919021395420160/0)

  为什么GitHub需要SSH Key呢？因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。

  当然，GitHub允许你添加多个Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。

  最后友情提示，在GitHub上免费托管的Git仓库，任何人都可以看到喔（但只有你自己才能改）。所以，不要把敏感信息放进去。关联完成之后就可以向远程仓库推送了。

  

  接下来介绍一些常用的操作

  ```git
  git log --pretty=oneline            将commit信息压缩成一行
  git reset --hard head^              返回上一个版本
  git reset --haed commitID           返回指定commitID所指向的版本
  git reflog                          记录每一次的操作命令
  git checkout -- file                丢弃工作区的修改，如果是添加到暂存区后还做了修改那                                     么此命令将会把文件恢复到初始添加到暂存区时的状态
  git reset head file                 将已经放入到暂存区的文件重新放回工作区
  git rm file                         文件手动删除之后再使用此命令将彻底删除此文件
  git checkout -- file                从版本库里恢复误删文件
  git branch branchname               创建branchname分支
  git checkout branchname             切换到branchname分支
  git checkout -b branchname          相当于上两条命令
  git branch -d branchname            删除branchname分支
  git merge branchname                合并branchname分支到当前分支
  git log --graph                     查看分支合并图
  git stash                           保存当前修改
  git cherry-pick commitID            将特定分支的提交复用到当前分支
  git remote                          查看远程库信息
  git remote -v                       查看自己对远程仓库所拥有的权限
  git checkout -b dev origin/dev      如果从远程仓库克隆的是master分支使                                     用此命令可以切换到远程dev分支，但是                                     此时可能并未指定当前分支与远程dev分                                     支关联，需要使用此命令进行关联  git                                     branch --set-upstream-                                             to=origin/dev dev               
  git log --pretty=oneline --abbrev-commit     和第一句差不多，但是这个                                     将commitID也压缩了
  git tag tagname                     将最新的commitID值变为tagname
  git tag tagname commitID            将指定commitID值变更为tagname
  git show tagname                    显示tag信息
  git tag -d tagname                  删除tag
  
  ```

  

* 

* 


# FastJSON

```java
Stringtestjsonstr=son.testString;
//json字符串转json对象
JSONObjectjsonObject=JSONObject.parseObject(testjsonstr);
//System.out.println(jsonObject.get("message"));
//获取json对象里的json对象
JSONObject jsonObjectone=(JSONObject)jsonObject.get("data");
JSONObject jsonObjecttwo=jsonObject.getJsonObject("cityInfo");
//System.out.println(jsonObjectone.get("quality"));
//获取json对象里的json对象数组,获取完后可以像数组那样直接获取
JSONArrayjsonArray=jsonObjectone.getJSONArray("forecast");
//System.out.println(jsonArray.get(1));
//json对象转普通对象
//构建json结构数据
JSONObjectjsonObjectres=newJSONObject();
jsonObjectres.put("message",jsonObject.get("message").toString());
jsonObjectres.put("city",jsonObjecttwo.get("city").toString());
jsonObjectres.put("citykey",jsonObjecttwo.get("citykey").toString());
//将数据转换为目标类
POJOpojo=JSONObject.parseObject(jsonObjectres.toJSONString(),POJO.class);
System.out.println(pojo.getCity());
```

# MongoDB

数据库在很大程度上是由其数据模型来定义的。MongoDB的数据模型是面向文档的。文档基本上是一组属性名和属性值的集合。属性的值可以是简单的数据类型，例如字符串、数字和日期。但这些值也可以是数组，甚至是其他文档，这让文档可以表示各种富数据结构。相对于传统的关系型数据库，MongoDB无须预定义Schema，这就意味着当开发初期需要频繁修改字段的话工作量大大减少。

## 创建/删除数据库

```
> use databasename
> db.dropdatabase()
```

此时数据库并未真正的创建成功，当第一次插入数据的时候才会创建成功。

## 创建/删除集合

```
> db.createCollection(name, options)
                                        参数说明：
                                        name: 要创建的集合名称
                                        options: 可选参数, 指定有关内存大小及索引的选项

还可以直接插入集合数据，Mongo会帮你自动创建集合
> db.mycol2.insert({"name" : "菜鸟教程"})
> show collections
  mycol2
> db.collection.drop()                  删除集合
```

## 插入文档

```
> db.col.insert(
    {   
        title: 'MongoDB 教程', 
        description: 'MongoDB 是一个 Nosql 数据库',
        by: '菜鸟教程',
        url: 'http://www.runoob.com',
        tags: ['mongodb', 'database', 'NoSQL'], //注意这里可以插入数组，这点和普通json不一样
        likes: 100
    }
)
```

## 更新文档

```
> db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})
以上语句只会修改第一条发现的文档，如果你要修改多条相同的文档，则需要设置 multi 参数为 true。
> db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
> db.collection.save()  //这条命令和insert不一样的是insert会检查id是否冲突，冲突了则不插                         //入，save命令不管，有冲突也插进去当成更新操作，没有冲突的话就和                           
      //insert一样
```

## 删除文档

```
db.col.remove({"title":"MongoDB 教程"})   //删除符合条件的文档
db.col.remove({"title":"MongoDB 教程"},1) //仅删除一条
```

## 查询文档

```
db.col.find()
```

## 条件操作符

### MongoDB (>) 大于操作符 - $gt

如果你想获取 "col" 集合中 "likes" 大于 100 的数据，你可以使用以下命令：

```
db.col.find({likes : {$gt : 100}})
```

类似于SQL语句：

```
Select * from col where likes > 100;
```

### MongoDB（>=）大于等于操作符 - $gte

如果你想获取"col"集合中 "likes" 大于等于 100 的数据，你可以使用以下命令：

```
db.col.find({likes : {$gte : 100}})
```

类似于SQL语句：

```
Select * from col where likes >=100;
```

### MongoDB (<) 小于操作符 - $lt

如果你想获取"col"集合中 "likes" 小于 150 的数据，你可以使用以下命令：

```
db.col.find({likes : {$lt : 150}})
```

类似于SQL语句：

```
Select * from col where likes < 150;
```

### MongoDB (<=) 小于等于操作符 - $lte

如果你想获取"col"集合中 "likes" 小于等于 150 的数据，你可以使用以下命令：

```
db.col.find({likes : {$lte : 150}})
```

类似于SQL语句：

```
Select * from col where likes <= 150;
```

### MongoDB 使用 (<) 和 (>) 查询 - $lt 和 $gt

如果你想获取"col"集合中 "likes" 大于100，小于 200 的数据，你可以使用以下命令：

```
db.col.find({likes : {$lt :200, $gt : 100}})
```

类似于SQL语句：

```
Select * from col where likes>100 AND  likes<200;
```

### MongoDB 操作符 - $type 实例

如果想获取 "col" 集合中 title 为 String 的数据，你可以使用以下命令：

```
db.col.find({"title" : {$type : 2}})
或
db.col.find({"title" : {$type : 'string'}})
```

### MongoDB Limit与Skip方法

limit()方法基本语法如下所示：

```
>db.COLLECTION_NAME.find().limit(NUMBER)
//示例
> db.col.find({},{"title":1,_id:0}).limit(2)
{ "title" : "PHP 教程" }
{ "title" : "Java 教程" }
```

skip() 方法脚本语法格式如下：

```
>db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
//实例以下实例只会显示第二条文档数据
>db.col.find({},{"title":1,_id:0}).limit(1).skip(1)
{ "title" : "Java 教程" }
```

### MongoDB 排序

在 MongoDB 中使用 sort() 方法对数据进行排序，sort() 方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而 -1 是用于降序排列。

sort()方法基本语法如下所示：

```
>db.COLLECTION_NAME.find().sort({KEY:1})
```





# Springboot整合JPA

## 环境准备

* 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

* application.properties配置信息

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
```

> ddl-auto

- `create`：每次运行程序时，都会重新创建表，故而数据会丢失
- `create-drop`：每次运行程序时会先创建表结构，然后待程序结束时清空表
- `upadte`：每次运行程序，没有表时会创建表，如果对象发生改变会更新表结构，原有数据不会清空，只会更新（推荐使用）
- `validate`：运行程序会校验数据与数据库的字段类型是否相同，字段不同会报错
- `none`: 禁用DDL处理

## 简单的REST CRUD示例

* 实体类

```java
package com.example.springbootjpa.entity;

import lombok.Data;
import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;

@Entity
@Table(name = "tb_user")
@Data
public class User {

    @Id
    @GenericGenerator(name = "idGenerator", strategy = "uuid")
    @GeneratedValue(generator = "idGenerator")
    private String id;

    @Column(name = "username", unique = true, nullable = false, length = 64)
    private String username;

    @Column(name = "password", nullable = false, length = 64)
    private String password;

    @Column(name = "email", length = 64)
    private String email;

}

```

> 主键采用UUID策略
>  `@GenericGenerator`是Hibernate提供的主键生成策略注解，注意下面的`@GeneratedValue`（JPA注解）使用generator = "idGenerator"引用了上面的name = "idGenerator"主键生成策略

* DAO层

```java
package com.example.springbootjpa.repository;

import com.example.springbootjpa.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, String> {
}
```

* Controller层

```java
package com.example.springbootjpa.controller;

import com.example.springbootjpa.entity.User;
import com.example.springbootjpa.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Optional;

@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserRepository userRepository;

    @PostMapping()
    public User saveUser(@RequestBody User user) {
        return userRepository.save(user);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable("id") String userId) {
        userRepository.deleteById(userId);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable("id") String userId, @RequestBody User user) {
        user.setId(userId);
        return userRepository.saveAndFlush(user);
    }

    @GetMapping("/{id}")
    public User getUserInfo(@PathVariable("id") String userId) {
        Optional<User> optional = userRepository.findById(userId);
        return optional.orElseGet(User::new);
    }

    @GetMapping("/list")
    public Page<User> pageQuery(@RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum,
                                @RequestParam(value = "pageSize", defaultValue = "10") Integer pageSize) {
        return userRepository.findAll(PageRequest.of(pageNum - 1, pageSize));
    }

}
```

## 使用

* 声明一个接口继承自Repository或Repositoy的一个子接口，对于Spring Data Jpa通常是JpaRepository，如：

```java
@RepositoryDefinition(domainClass = UserInfo.class, idClass = String.class)
public interface UserRepositry extends JpaRepository<UserInfo,String> {

    List<UserInfo> findByUsername(String username);

    List<UserInfo> findByEmail(String email);

    List<UserInfo> findByEmail(String email, Pageable pageable);
}
```

* 直接注入就行

```java
    @Autowired
    UserRepositry userRepositry;
```

## 分页查询及排序

Spring Data Jpa可以在方法参数中直接传入`Pageable`或`Sort`来完成动态分页或排序，通常Pageable或Sort会是方法的最后一个参数，如：

```java
@Query("select u from User u where u.username like %?1%")
Page<User> findByUsernameLike(String username, Pageable pageable);

@Query("select u from User u where u.username like %?1%")
List<User> findByUsernameAndSort(String username, Sort sort);
```

那调用repository方法时传入什么参数呢？对于Pageable参数，在Spring Data 2.0之前我们可以new一个`org.springframework.data.domain.PageRequest`对象，现在这些构造方法已经废弃，取而代之Spring推荐我们使用PageRequest的of方法

```java
new PageRequest(0, 5);
new PageRequest(0, 5, Sort.Direction.ASC, "username");
new PageRequest(0, 5, new Sort(Sort.Direction.ASC, "username"));
        
PageRequest.of(0, 5);               //这里切换到6-10页只需要将0自增成1
PageRequest.of(0, 5, Sort.Direction.ASC, "username");
PageRequest.of(0, 5, Sort.by(Sort.Direction.ASC, "username"));
```



# Maven多模块打包

尝试新建一个微服务项目并完成多模块打包部署首先项目结构如图所示，参考链接 

https://www.cnblogs.com/victorbu/p/10895676.html

![](E:\DistCode\TyporaLoad\Dubbo.assets\QQ截图20200728162149.jpg)

## 父元素pom信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>            //打包方式设置为pom
    <modules>                             //包含子模块
        <module>consumer</module>
        <module>provider</module>
        <module>Api</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0</version>
    <name>parent</name>
    <url>http://maven.apache.org</url>
    <description>Parent project for Spring Boot</description>
    
    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.plugin.version>3.6.0</maven.compiler.plugin.version>
        <mavne.surefire.plugin.version>2.19.1</mavne.surefire.plugin.version>
        <maven-war-plugin.version>2.6</maven-war-plugin.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <!--============== 配置私服START =============== -->
    <repositories>
        <repository>
            <id>DistNexus</id>
            <url>http://58.246.138.178:22280/nexus/content/groups/public/</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>DistNexus</id>
            <url>http://58.246.138.178:22280/nexus/content/groups/public/</url>
        </pluginRepository>
    </pluginRepositories>
    <distributionManagement>
        <repository>
            <id>DistNexusRelease</id>
            <url>http://58.246.138.178:22280/nexus/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>DistNexusSnapshot</id>
            <url>http://58.246.138.178:22280/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
    <!--============== 配置私服End =============== -->
    
    <!-- 打包编译插件 -->
    
    <build>
        <plugins>
            <!--maven的编译插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven.compiler.plugin.version}</version>
                <configuration>
                    <!--开发版本-->
                    <source>${java.version}</source>
                    <!--.class文件版本-->
                    <target>${java.version}</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>
            <!--maven打包跳过测试-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${mavne.surefire.plugin.version}</version>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Api模块pom配置信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0</version>                     <!-- 依赖父pom，必须指定父pom的地址！ -->
        <relativePath>../pom.xml</relativePath><!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>api</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>api</name>
    <description>Api project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>

    <build>
        <finalName>Api</finalName>
    </build>

</project>
```

## consumer模块pom信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0</version>                     <!-- 依赖父pom，必须指定父pom的地址！ -->
        <relativePath>../pom.xml</relativePath> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>consumer</artifactId>
    <version>1.0.0</version>
    <packaging>war</packaging>
    <name>consumer</name>
    <description>consumer project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>api</artifactId>
            <version>1.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>

<!-- 必要的配置，指定主类以及springboot自带的打包插件 -->
    <build>
        <finalName>consumer</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <!--创建项目时自带的 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- 自己添加的 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>1.5.13.RELEASE</version>
                <configuration>
                    <mainClass>
                        com.example.consumer.ConsumerApplication
                    </mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

## provider模块的pom信息（和consumer几乎一样）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0</version>
        <relativePath>../pom.xml</relativePath> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>provider</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>provider</name>
    <description>provider project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>api</artifactId>
            <version>1.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>

    <build>
        <finalName>provider</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <!--创建项目时自带的 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- 自己添加的 -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>1.5.13.RELEASE</version>
                <configuration>
                    <mainClass>
                        com.example.provider.ProviderApplication
                    </mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>

```

全部完成之后依次点击父模块的clean->install完成打包



# Springboot集成Swagger



## 加入依赖

```xml
	<dependency>
		<groupId>com.spring4all</groupId>
		<artifactId>swagger-spring-boot-starter</artifactId>
		<version>1.7.0.RELEASE</version>
	</dependency>
```

## 启动类加上注解

```java
import io.swagger.annotations.Api;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@EnableSwagger2
@SpringBootApplication
@MapperScan("com.example.testspringboot.dao")
public class DemoApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
    @Override//为了打包springboot项目
    protected SpringApplicationBuilder configure(
            SpringApplicationBuilder builder) {
        return builder.sources(this.getClass());
    }

}
```

## application.properties文件配置访问地址

```java
springfox.documentation.swagger.v2.path: /api-docs

默认访问地址是：http://localhost:8080/swagger-ui.html

可以通过这种方式修改路径:
    
@Controller
public class HomeController {

    @RequestMapping(value = "/swagger")
    public String index() {
        System.out.println("swagger-ui.html");
        return "redirect:swagger-ui.html";
    }
}
```

## 注解一览

### @Api标记

```JAVA
//Api 用在类上，说明该类的作用。可以标记一个Controller类做为swagger 文档资源，使用方式：

@Api(value = "/user", description = "Operations about user")

@Controller
@RequestMapping("/controller1")
@Api(value = "/user", description = "Operations about user")
public class SwaggerController {
```

### @ApiOperation

```java
//ApiOperation：用在方法上，说明方法的作用，每一个url资源的定义,使用方式：

@ApiOperation(
          value = "Find purchase order by ID",
          notes = "For valid response try integer IDs with value <= 5 or > 10. Other values will generated exceptions",
          response = Order,
          tags = {"Pet Store"})
```

@ApuParam

```java
//ApiParam请求属性,使用方式:

public ResponseEntity<Order> getOrderById(
      @ApiParam(value = "ID of pet that needs to be fetched", allowableValues = "range[1,5]", required = true)
      @PathVariable("orderId") String orderId)
```

### @ApiModel

描述一个Model的信息（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候；

### @ApiModelProperty

描述一个model的属性。







