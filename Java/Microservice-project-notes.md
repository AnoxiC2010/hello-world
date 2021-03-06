# 微服务项目



https://blog.csdn.net/weixin_43770545/article/details/90486809

```
com.mysql.cj.jdbc.Driver


com.mysql.jdbc.Driver和mysql-connector-java 5一起用。
com.mysql.cj.jdbc.Driver和mysql-connector-java 6 一起用。
```



日志

lombok的

@Sl4j注解提供log属性

线程debug模式

swagger注解

select for update

CollectionUtils springframeword的

全局id生成器 （发号器 实现生成唯一id的工具）

JWT 生成与取出

图片验证码

防止跨域问题的配置

redis集群 

项目微服务集群

消息中间件集群

数据库分表



spring事务生效和不生效情况，处理



## 接口的幂等性

分布式项目接口的幂等性设计



遇到问题：

微服务rpc调用，insert语句执行插入了



## 下单

买家留言暂不实现

评价暂不实现

unique key唯一键 代替订单id暴漏给外部



商品库存售卖id 不同区域有不同的库存

![image-20210619102039646](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619102039646.png)





![image-20210619112009978](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619112009978.png)

pipeline采用工厂方法来创建

![image-20210619112152271](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619112152271.png)

这个工厂方法

![image-20210619114159615](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619114159615.png)

![image-20210619144623741](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619144623741.png)

# Pipeline设计模式

- 简介

  Pipeline模式⼜称为流⽔线模式，pipeline⼜称为管道，是⼀种在计算机普遍使⽤的技术。举个最普遍的例⼦，如下图所示cpu流⽔线，⼀个流⽔线分为4部分，每个部分可以独⽴⼯作，于是可以处理多个数据流。linux 管道也是⼀个常⽤的管道技术，其字符处理功能⼗分强⼤，在⾯试过程中常会被问到。在分布式处理领域，由于管道模式是数据驱动，⽽⽬前流⾏的Spark分布式处理平台也是数据驱动的，两者⾮常合拍，于是在spark的新的api⾥⾯pipeline模式得到了⼴泛的应⽤。还有java web中的struct的filter、netty的pipeline，⽆处不⻅pipeline模式。
  Netty：NIO框架，Dubbo底层调⽤的时候就是⽤的netty框架（Mina）

- 解决的问题：

  有时⼀些线程的步骤⽐较冗⻓，⽽且由于每个阶段的结果与下阶段的执⾏有关系，⼜不能分开

- 解决思路

  可以将任务的处理分解为若⼲个处理阶段，上⼀个阶段任务的结果交给下⼀个阶段来处理，这样每个线程的处理是并⾏的，可以充分利⽤资源提⾼计算效率。



### 概念

管道模型包含两个部分：pipeline 管道、valve 阀⻔（也称为handler）、Context（返回结果的上下⽂）。

pipeline 管道，可以⽐作⻋间⽣产线，在这⾥可认为是容器的逻辑处理总线。
valve 阀⻔，可以⽐作⽣产线上的⼯⼈，负责完成各⾃的部分⼯作。 阀⻔也可以叫做Handler 处理者

![image-20210620221703975](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210620221703975.png)



### 接⼝代码建模

Handler接⼝

```JAVA
public interface Handler {
    Boolean handle(MyContext context);
}
```

操作节点单元

```JAVA
@Data
public class HandlerNode {

    private Handler handler;

    private HandlerNode next;

    public void exec(MyContext context){
        Boolean ret = handler.handle(context);
        if (ret) {
            if (next != null) {
                next.exec(context);
            }
        }
    }
}
```

上下⽂环境

```JAVA
@Data
public class PipelineContext {
 String orderId;
}
```

管道接⼝

```java
/**
* 管道
* 其中的"PipelineContext"俗称“上下⽂”，
* 代表从开始贯穿到结尾的⼀个执⾏环境，⽤于在各个Pipeline之间传递信息；
* ⼀个“Pipeline”包含两个⽅法，
* process代表当前的处理流程，
* forward⽅法代表将处理消息转发给下游的流程：上游可以控制消息是否转发给下游。
*/
public interface Pipeline {
 /**
 * 启动流程
 */
 void start();
 /**
 * 终⽌流程
 */
 void shutdown();
 /**
 * 获取返回值
 */
 PipelineContext getContext();
 /**
 * 添加到头部
 * @param handlers
 */
 void addFirst(Handler ... handlers);
 /**
 * 添加到尾部
 */
 void addLast(Handler ... handlers);
}
```

管道对象实现

```java
public class OrderPipeline implements Pipeline {
    //尾部节点
    private OrderHandlerNode tail;
    //头部节点
    private OrderHandlerNode head = new OrderHandlerNode();
    // 上下⽂环境
    private PipelineContext context = null;
    public OrderPipeline(PipelineContext context) {
        this.context = context;
    }
    /**
 * 添加到队⾸
 * @param handlers
 */
    public void addFirst(Handler... handlers) {
        OrderHandlerNode pre = head.getNext();
        for (Handler handler : handlers) {
            if (handler == null) {
                continue;
            }
            OrderHandlerNode orderHandlerNode = new OrderHandlerNode();
            orderHandlerNode.setHandler(handler);
            orderHandlerNode.setNext(pre);
            pre = orderHandlerNode;
        }
        head.setNext(pre);
    }
    /**
 * 添加到队尾
 * @param handlers
 */
    public void addLast(Handler... handlers) {
        OrderHandlerNode next = tail;
        for (Handler handler : handlers) {
            if (handler == null) {
                continue;
            }
            OrderHandlerNode orderHandlerNode = new OrderHandlerNode();
            orderHandlerNode.setHandler(handler);
            next.setNext(orderHandlerNode);
            next = orderHandlerNode;
        }
        tail = next;
    }
    /**
 * 启动流程
 */
    public void start() {
        head.getNext().exec(getContext());
    }
    /**
 * 获取context
 * @param
 * @return
 */
    public PipelineContext getContext() {
        return context;
    }
}
```

注意事项

1. 阀⻔的实现分两种，即普通阀⻔和尾阀⻔。普通阀⻔在处理完⾃⼰的事情之后，必须调⽤
getNext.invoke(s)⽅法，也就是交给下⼀个阀⻔处理；⽽尾阀⻔不⽤调⽤这个⽅法，因为它是结束的那个阀⻔。
2. 流⽔线实现类的主要逻辑在addFirst和addLast，它持有⼀个头阀⻔和⼀个尾阀⻔，它按照添加阀⻔顺序的⽅式构造阀⻔链表，按照队列的形式，决定调⽤的先后顺序
3. Pipeline的深度:Pipeline中handler的个数被称作Pipeline的深度。所以我们在⽤Pipeline的深度与JVM宿主机的CPU个数间的关系。如果Pipeline实例所处的任务多属于CPU密集⾏，那么深度最好不超过Ncpu。如果Pipeline所处理的任务多属于I/O密集型，那么Pipeline的深度最好不要超过2*Ncpu。



优点

总结⼀下Pipeline模式的优点，如下：
1、降低耦合度。它将请求的发送者和接收者解耦。
2、简化了Handler处理器。使得处理器不需要不需要知道链的结构。也就是Handler处理器可以是⽆状态的。与责任链（流⽔线）相关的状态，交给了Context去维护。
3、增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。
4、 增加新的请求处理器很⽅便。





并发编程 Doug Lee

AbstractQueuedSynchronizer(java.util.concurrent.locks)  AQS

抽象的队列同步器

链表把头部设置为空节点，方便插入，链表里的基本的思想

抽象的队列同步器作为模板方法提供了很多实现类

实现类

公平锁、非公平锁、可重入锁、计数器、信号量、栅栏...

![image-20210619104329049](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619104329049.png)





链表把头部设置为空节点，方便插入，链表里的基本的思想

有了头部空节点，插入方便，空节点永远是头部

![image-20210619105206968](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619105206968.png)

![image-20210619105605278](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619105605278.png)



# ![image-20210619110146640](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210619110146640.png)



## 管道模式应用



### 创建订单接⼝业务分析

创建订单接⼝需要做什么事情呢？
1. 参数校验
2. 向订单表中插⼊⼀条数据
3. 向订单商品关联表中插⼊N条数据
4. 扣减库存
5. 向订单邮寄表中插⼊⼀条数据
6. 清空购物⻋
7. 发送订单超时取消的消息（暂时不做）

我们发现创建⼀个订单的流程很⻓，步骤⽐较冗⻓，⽽且每⼀步的执⾏结果都会影响到下⼀步的操作，所以我们这⾥采取pipeline模式来设计整体架构



### 订单服务表分析

订单表

```sql
CREATE TABLE `tb_order` (
  `order_id` varchar(50) COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT '订单id',
  `payment` decimal(10,2) DEFAULT NULL COMMENT '实付金额',
  `payment_type` int(1) DEFAULT NULL COMMENT '支付类型 1在线支付 2货到付款',
  `post_fee` decimal(10,2) DEFAULT NULL COMMENT '邮费',
  `status` int(1) DEFAULT NULL COMMENT '状态 0未付款 1已付款 2未发货 3已发货 4交易成功 5交易关闭 6交易失败 7-已退款',
  `create_time` datetime DEFAULT NULL COMMENT '订单创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '订单更新时间',
  `payment_time` datetime DEFAULT NULL COMMENT '付款时间',
  `consign_time` datetime DEFAULT NULL COMMENT '发货时间',
  `end_time` datetime DEFAULT NULL COMMENT '交易完成时间',
  `close_time` datetime DEFAULT NULL COMMENT '交易关闭时间',
  `shipping_name` varchar(20) COLLATE utf8_bin DEFAULT NULL COMMENT '物流名称',
  `shipping_code` varchar(20) COLLATE utf8_bin DEFAULT NULL COMMENT '物流单号',
  `user_id` bigint(20) DEFAULT NULL COMMENT '用户id',
  `buyer_message` varchar(100) COLLATE utf8_bin DEFAULT NULL COMMENT '买家留言',
  `buyer_nick` varchar(50) COLLATE utf8_bin DEFAULT NULL COMMENT '买家昵称',
  `buyer_comment` tinyint(1) DEFAULT NULL COMMENT '买家是否已经评价',
  `unique_key` varchar(50) COLLATE utf8_bin DEFAULT NULL COMMENT '唯一键',
  PRIMARY KEY (`order_id`),
  KEY `create_time` (`create_time`),
  KEY `buyer_nick` (`buyer_nick`),
  KEY `status` (`status`),
  KEY `payment_type` (`payment_type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```

订单商品关联表

```sql
CREATE TABLE `tb_order_item` (
  `id` varchar(50) COLLATE utf8_bin NOT NULL,
  `item_id` bigint(20) NOT NULL COMMENT '商品id',
  `order_id` varchar(50) COLLATE utf8_bin NOT NULL COMMENT '订单id',
  `num` int(10) DEFAULT NULL COMMENT '商品购买数量',
  `title` varchar(200) COLLATE utf8_bin DEFAULT NULL COMMENT '商品标题',
  `price` decimal(10,2) DEFAULT NULL COMMENT '商品单价',
  `total_fee` decimal(10,2) DEFAULT NULL COMMENT '商品总金额',
  `pic_path` varchar(1000) COLLATE utf8_bin DEFAULT NULL COMMENT '商品图片地址',
  `status` int(4) DEFAULT NULL COMMENT '1库存已锁定 2库存已释放 3-库存减扣成功',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `oder_item_id` (`order_id`,`item_id`) USING BTREE COMMENT '订单商品唯一索引',
  KEY `item_id` (`item_id`),
  KEY `order_id` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```

商品库存表

```java
CREATE TABLE `tb_stock` (
  `item_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '商品id',
  `stock_count` bigint(20) NOT NULL DEFAULT '0' COMMENT '库存数量',
  `lock_count` int(11) NOT NULL DEFAULT '0' COMMENT '冻结库存数量',
  `restrict_count` int(3) DEFAULT '5' COMMENT '限购数量',
  `sell_id` int(6) DEFAULT NULL COMMENT '售卖id',
  PRIMARY KEY (`item_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='库存表';
```

物流信息表

```sql
CREATE TABLE `tb_order_shipping` (
  `order_id` varchar(50) NOT NULL COMMENT '订单ID',
  `receiver_name` varchar(20) DEFAULT NULL COMMENT '收货人全名',
  `receiver_phone` varchar(20) DEFAULT NULL COMMENT '固定电话',
  `receiver_mobile` varchar(30) DEFAULT NULL COMMENT '移动电话',
  `receiver_state` varchar(10) DEFAULT NULL COMMENT '省份',
  `receiver_city` varchar(10) DEFAULT NULL COMMENT '城市',
  `receiver_district` varchar(20) DEFAULT NULL COMMENT '区/县',
  `receiver_address` varchar(200) DEFAULT NULL COMMENT '收货地址，如：xx路xx号',
  `receiver_zip` varchar(6) DEFAULT NULL COMMENT '邮政编码,如：310001',
  `created` datetime DEFAULT NULL,
  `updated` datetime DEFAULT NULL,
  PRIMARY KEY (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



# ⽤户服务业务分析

## 1.1 表结构设计

⽤户表：

```sql
CREATE TABLE `tb_member` (
 `id` bigint(20) NOT NULL AUTO_INCREMENT,
 `username` varchar(50) NOT NULL COMMENT '⽤户名',
 `password` varchar(32) NOT NULL COMMENT '密码，加密存储',
 `phone` varchar(20) DEFAULT NULL COMMENT '注册⼿机号',
 `email` varchar(50) DEFAULT NULL COMMENT '注册邮箱',
 `created` datetime NOT NULL,
 `updated` datetime NOT NULL,
 `sex` varchar(2) DEFAULT '',
 `address` varchar(255) DEFAULT NULL,
 `state` int(1) DEFAULT '0',
 `file` varchar(255) DEFAULT NULL COMMENT '头像',
 `description` varchar(500) DEFAULT NULL,
 `points` int(11) DEFAULT '0' COMMENT '积分',
 `balance` decimal(10,2) DEFAULT '0.00' COMMENT '余额',
 `isverified` varchar(26) DEFAULT 'N',
 PRIMARY KEY (`id`),
 UNIQUE KEY `username` (`username`) USING BTREE,
 UNIQUE KEY `phone` (`phone`) USING BTREE,
 UNIQUE KEY `email` (`email`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=78 DEFAULT CHARSET=utf8 COMMENT='⽤户表';
```

⽤户认证表：

```sql
CREATE TABLE `tb_user_verify` (
 `id` bigint(20) NOT NULL AUTO_INCREMENT,
 `username` varchar(56) DEFAULT NULL,
 `register_date` datetime DEFAULT NULL,
 `uuid` varchar(56) DEFAULT NULL,
 `is_verify` varchar(10) DEFAULT NULL COMMENT '是否验证Y已验证，N未验证',
 `is_expire` varchar(255) DEFAULT NULL COMMENT '是否过期Y已过期，N未过期',
 PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

## 1.2 业务分析

- ⽤户注册接⼝
  1. 验证传过来的验证码
  2. 向⽤户表中插⼊⼀条记录
  3. 向⽤户验证表中插⼊⼀条记录
  4. 发送⽤户激活邮件
- ⽤户登录接⼝
   1. 验证验证码
   2. 验证⽤户名以及密码
   3. 产⽣⼀个合法的JWT

1.3 发送邮件



# java发送邮件

简介
要实现邮件发送与接收功能，必须要有专⻔的邮件服务器。这些邮件服务器类似于现实⽣活中的邮局，它主要负责接收⽤户投递过来的邮件，并把邮件投递到邮件接收者的电⼦邮箱中。

SMTP服务器地址：⼀般是 smtp.xxx.com，⽐如163邮箱是smtp.163.com，qq邮箱是smtp.qq.com。



流程图：

![image-20210624090811921](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210624090811921.png)



sun公司给我们提供了Java发送邮件的解决⽅案，并且提供了jar包

```xml
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
</dependency>
```

注意事项

这⾥以163邮箱为例，qq邮箱也是⼀样的。需要把邮箱的smtp服务打开

![image-20210624090902791](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210624090902791.png)

管理授权码，也就是发送的时候需要⽤到的密码

![image-20210624090927445](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210624090927445.png)



⽽Springboot项⽬则给我们提供了封装了上⾯mail包的接⼝，让我们在Springboot项⽬中能够更⽅便的 实现邮件发送

- 导包

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-mail</artifactId>
  </dependency>
  ```

- 配置

  ```properties
  #邮件服务器
  spring.mail.host=smtp.163.com
  #端⼝
  spring.mail.port=25
  #发送邮件邮箱⽤户名
  spring.mail.username=ciggarnot@163.com
  #发送邮件邮箱密码（这个密码是上⾯的授权码，不是邮箱登录的密码）
  spring.mail.password=SNTGFTIEAJUMRHCT
  #设置邮箱认证打开
  spring.mail.properties.mail.smtp.auth=true
  ```

  

- 代码编写

  ```java
  @Autowired
  private JavaMailSender javaMailSender;
  @Test
  public void sendMail(){
      SimpleMailMessage message = new SimpleMailMessage();
      message.setFrom("ciggarnot@163.com");
      message.setTo("291136733@qq.com");
      message.setSubject("商城激活");
      message.setText("http://localhost:8080/user/verify?uid=akjsdnh39udklnw");
      javaMailSender.send(message);
  /*
  javaMailSender.send(MimeMessage mimeMessage)发送有附件的邮件
  javaMailSender.send(SimpleMailMessage simpleMailMessage)发送简单的邮件对象
  */
  }
  ```

当然还有其他的邮件发送工具，比如Hutools工具包也提供方便的发送邮件功能



# 秒杀业务

## 1 表结构分析

秒杀场次表

```sql
CREATE TABLE `tb_promo_session` (
 `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
 `session_id` int(4) NOT NULL COMMENT '场次 id 1:上午⼗点场 2:下午四点场',
 `start_time` datetime NOT NULL COMMENT '开始时间',
 `end_time` datetime NOT NULL COMMENT '结束时间',
 `yyyymmdd` varchar(255) NOT NULL COMMENT '场次⽇期',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

场次商品关联表

```sql
CREATE TABLE `tb_promo_item` (
 `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
 `ps_id` int(11) NOT NULL COMMENT '秒杀场次id',
 `item_id` int(11) NOT NULL COMMENT '商品id',
 `seckill_price` decimal(10,2) NOT NULL COMMENT '商品秒杀价格',
 `item_stock` int(11) NOT NULL COMMENT '商品秒杀库存',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
```



## 2 接⼝⽂档

### 2.1 获取秒杀商品列表接⼝

⼊参 类型 含义

```
sessionId Integer 场次id 1:上午10点场 2:下午4点场
```

请求路径：/shopping/seckilllist
是否需要携带Token：否
请求类型：get
请求参数示例：
⽆

出参 类型 含义

```
success boolean 成功标记
message String 具体信息
code Integer 状态码
result String(JSON) 具体数据
timestamp Long 时间戳
```



返回参数示例：

```json
{ "success":true, "message":"success", "code":200, "result":{ "code":"000000", "msg":"成功",
"sessionId":1, "psId":1, "productList":[ { "id":100057401, "inventory":10, "price":149,
"seckillPrice":49, "picUrl":"https://resource.smartisan.com/resource/005c65324724692f7c9b
a2fc7738db13.png", "productName":"Smartisan T恤 迪特拉姆斯" }, { "id":100052801,
"inventory":7, "price":149, "seckillPrice":199, "picUrl":"https://resource.smartisan.com/reso
urce/6e96ccea3bd56bdd2243eb20330cec30.jpg", "productName":"坚果砖式蓝⽛⼩⾳箱" } ] },
"timestamp":1588535951303 }
```



### 2.2 获取秒杀商品详情接⼝

⼊参 类型 含义

```
psId Long 秒杀场次主键id
productId Long 商品id
```

请求路径：/shopping/promoProductDetail
是否需要携带Token：是
请求类型：post
请求参数示例：
{"productId":100057401,"psId":2}

出参 类型 含义

```
success boolean 成功标记
message String 具体信息
code Integer 状态码
result String(JSON) 具体数据
timestamp Long 时间戳
```



返回参数示例：

```json
{ "success":true, "message":"success", "code":200, "result":{ "code":"000000", "msg":"成功",
"promoProductDetailDTO":{ "productId":100057401, "salePrice":149,
"productName":"Smartisan T恤 迪特拉姆斯", "subTitle":"", "limitNum":100,
"productImageBig":"https://resource.smartisan.com/resource/005c65324724692f7c9ba2fc7
738db13.png", "detail":"xxx", "productImageSmall":[ "https://resource.smartisan.com/reso
urce/005c65324724692f7c9ba2fc7738db13.png", "https://resource.smartisan.com/resource/
5068afef4f8866fae065d8c0d450e244.png", "https://resource.smartisan.com/resource/a8dfe
8f52dfb15c17e2e5c504c7ae2c6.png", "https://resource.smartisan.com/resource/d6a6c06e5
b51f0c18d8bfc45318163ea.png", "https://resource.smartisan.com/resource/46724a81b037d
1eca31d665c223b77a1.png" ], "promoPrice":49 } }, "timestamp":1592288092856 }
```



### 2.3 秒杀下单接⼝(⽣成订单、扣减库存、⽣成商品的邮寄信息)

⼊参 类型 含义

```
psId Long 秒杀场次主键id
productId Long 商品id
addressId Long 地址id
tel String ⼿机号码
streetName String 地址
userName String ⽤户名
```

请求路径：/shopping/seckill
是否需要携带Token：是
请求类型：post
请求参数示例：
⽆

出参 类型 含义

```
success boolean 成功标记
message String 具体信息
code Integer 状态码
result String(JSON) 具体数据
timestamp Long 时间戳
```

返回参数示例：

```json
{ "success":true, "message":"success", "code":200, "result":{ null },
"timestamp":1588535951303 }
```

### 3 代码编写

...



# 分布式事务

## 1 知识回顾

### 1.1 ⽬前秒杀模型

![image-20210625091357014](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625091357014.png)

### 1.2 存在的问题

问题：假如扣减秒杀库存成功，⽽⽣成订单失败了，那么扣减秒杀库存能回滚吗？
如果不能，原因是什么
改如何改进呢？



## 2 传统事务回顾

### 2.1 事务定义

事务定义：
是数据库操作的最⼩⼯作单元，是作为单个逻辑⼯作单元执⾏的⼀系列操作；这些操作作为⼀个整体⼀起向系统提交，要么都执⾏、要么都不执⾏



### 2.2 传统事务知识点

#### 2.2.1 四个特性（ACID）

- 原⼦性：事务是数据库的逻辑⼯作单位，事务中包含的各操作要么都做，要么都不做
- ⼀致性：事务执⾏的结果必须是使数据库从⼀个⼀致性状态变到另⼀个⼀致性状态。因此当数据库只包含成功事务提交的结果时，就说数据库处于⼀致性状态。如果数据库系统运⾏中发⽣故障，有些事务尚未完成就被迫中断，这些未完成事务对数据库所做的修改有⼀部分已写⼊物理数据库，这时数据库就处于⼀种不正确的状态，或者说是 不⼀致的状态。
- 隔离性：⼀个事务的执⾏不能其它事务⼲扰。即⼀个事务内部的操作及使⽤的数据对其它并发事务是隔离的，并发执⾏的各个事务之间不能互相⼲扰。
- 持久性：⼀个事务⼀旦提交，它对数据库中的数据的改变就应该是永久性的。接下来的其它操作或故障不应该对其执⾏结果有任何影响。



#### 2.2.2 事务隔离级别

- 读未提交 (Read Uncommitted)：允许脏读，也就是可能读取到其他会话中未提交事务修改的数据
- 读已提交(Read Committed)：只能读取到已经提交的数据。Oracle等多数数据库默认都是该级别(不重复读)
- 可重复读(Repeated Read)：在同⼀个事务内的查询都是事务开始时刻⼀致的，InnoDB默认级别。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻象读，但是innoDB解决了幻读
- 序列化：(Serializable)：完全串⾏化的读，每次读都需要获得表级共享锁，读写相互都会阻塞



#### 2.2.3 事务的传播⾏为

Spring ⽀持 7 种事务传播⾏为：
org.springframework.transaction.annotation. Propagation

![image-20210625091733877](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625091733877.png)

Propagation.REQUIRED 是常⽤的事务传播⾏为，如果当前没有事务，就新建⼀个事务，如果已经存
在⼀个事务中，加⼊到这个事务中。其它传播⾏为⼤家可另查阅。



#### 2.2.4 传统事务引⽤场景举例

![image-20210625091847813](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625091847813.png)

![image-20210625091908377](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625091908377.png)

传统事务控制基础：必须得保证是同⼀个连接，通过jdbc操作数据源的话保证同⼀个connection 对象



### 2.3 传统事务问题 

在分布式架构下，随着业务量的扩⼤，我们对业务进⾏拆分，数据库也会相应的进⾏分库分⽚，因为有 着⽹络的不确定性，那么我们分布式环境下应该如何保证事务的ACID？

![image-20210625092008683](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625092008683.png)

当单个数据库的性能产⽣瓶颈的时候，我们需要对数据库分库或者是分区，那么这个时候数据库就处于不同的服务器上了，因此基于单个数据库所控制的传统型事务已经不能在适应这种情况了，故我们需要使⽤分布式事务来管理这种情况



## 3 分布式事务

### 3.1 理论基础-CAP理论

#### 3.1.1 概念

1998年，加州⼤学的计算机科学家 Eric Brewer 提出，分布式系统有三个指标。

- ⼀致性：Consistency
  集群中各个结点的数据总是⼀致的，因此你可以向任意结点读写数据，并总是能得到相同的数据
- 可⽤性：Availability
  可⽤性表示你总是能够访问集群（读/写），即使集群中的某个结点宕机了
- 分区容忍性：Partition toleranc
  容忍集群持续运⾏，即使他们中存在分区（两个分区中的结点都是好的，只是分区之间不能通信)

![image-20210625092201961](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625092201961.png)



结论：分布式环境下，CAP不能同时成⽴



#### 3.1.2 模型解释

![image-20210625092255763](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625092255763.png)

1. 如上图所示，假如DB1和DB2都能够正常被访问，只是他们之间不能够互相通信，也就是他们之间不能够同步数据，这个时候我们容忍集群继续运⾏，那么我们说服务具有分区容忍性

2. 那么现在在分区容忍性的前提之下：（DB1和DB2不能通⾏，服务继续运⾏）

   现在向DB1中发送⼀条X=2的更新请求，Datebase1把X值更新为2，但是需要去同步到Database2，由于DB1和DB2不能够通⾏，所以同步会失败
   A, 假如该条请求返回更新成功，那么会导致DB1和DB2数据不⼀致的情况出现，违背了⼀致性
   B, 假如该条请求返回更新失败，那么我们认为DB1不可⽤了，违背了可⽤性

3. 分布式系统在什么时候存在不满⾜分区容忍性的情况呢？
容忍集群持续运⾏，即使他们中存在分区 (两个分区中的结点都是好的，只是分区之间不能通信)即P不存在，意思就是集群不能容忍分区的出现，也就是不能容忍两个节点之间不能通信的情况，⽽分布式环境下两个节点不能通信的情况是不存在的，因为节点之间的通信都是通过⽹络传输，⽹络是不“靠谱”的。



### 3.2 解决⽅案探索

#### 3.2.1 刚性事务（强⼀致性）

定义：遵循ACID原则，强⼀致性。
代表：⼆阶段提交（2PC）
⼆阶段提交协议是协调所有分布式原⼦事务参与者，并决定提交或取消（回滚）的分布式算法。

![image-20210625092553504](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625092553504.png)

引⼊了事务管理器
1. 预备阶段
在请求阶段，协调者将通知事务参与者准备提交或取消事务，然后进⼊表决过程。 在表决过程中，参与者将告知协调者⾃⼰的决策：同意（事务参与者本地作业执⾏成功）或取消（本地作业执⾏故障）。

2. 提交阶段
在该阶段，协调者将基于第⼀个阶段的投票结果进⾏决策：提交或取消。当且仅当所有的参与者同意提交事务协调者才通知所有的参与者提交事务，否则协调者将通知所有的参与者取消事务。参与者在接收到协调者发来的消息后将执⾏响应的操作。

存在的问题：
同步阻塞问题：在执⾏的过程中，所有参与的节点都是事务型阻塞的，当参与者占有公共资源时，其他第三⽅节点访问公共资源不得不处于阻塞状态
不能解决数据不⼀致的问题



#### 3.2.2 柔性事务（最终⼀致性）

**定义**
遵循BASE理论，最终⼀致性；与刚性事务不同，柔性事务允许⼀定时间内，不同节点的数据不⼀致，但
要求最终⼀致。

##### **Base理论**

- Basically Avaiable，基本可⽤
- Soft state：软状态
- Eventually consistent：最终⼀致性
  既然⽆法做到强⼀致性，但每个应⽤都可以根据⾃身的业务特点，采⽤适当的⽅式来使系统达到最终⼀致性。

##### **TCC事务**

全称：Try-Confirm-Cancel（可以理解为sql中的Lock、Commit、Rollback）
TCC是服务化的⼆阶段编程模型：
其 Try、Confirm、Cancel 3 个⽅法均由业务编码实现：Try 操作作为⼀阶段，负责资源的检查和预留。
Confirm 操作作为⼆阶段提交操作，执⾏真正的业务。Cancel 是预留资源的取消。

**事务流程**

![image-20210625095818498](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625095818498.png)

![image-20210625095836729](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625095836729.png)

- Try阶段：完成所有的业务检查，预留资源
- Confirm阶段：更改状态操作
- Cancel阶段：当Try阶段存在服务执⾏失败时，则进⼊Cancel阶段

![image-20210625095905497](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625095905497.png)



缺点：TCC的Try、Confirm、Cancel操作功能按照具体业务来实现，业务耦合度⾼，开发成本⾼

##### 本地消息表

本地消息表这个⽅案最初是ebay提出的分布式事务完整⽅案

![image-20210625100033142](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625100033142.png)

![image-20210625100045981](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625100045981.png)

##### MQ事务消息

![image-20210625100109382](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210625100109382.png)

1. 发送事务型消息，该消息暂时不可被消费
2. 消息队列返回发送消息成功状态
3. 本地事务收到消息发送成功状态，然后开始执⾏本地事务
  - 假如本地事务执⾏成功，MQ消息发送⽅则会告诉MQServer表示这个消息可以被消费了，MQ会投递这个消息到MQ订阅⽅
  - 假如本地事务执⾏失败，MQ消息发送⽅则会告诉MQServer需要丢弃之前发送的消息
4. 假如MQServer⼀直没有收到MQ发送⽅的消息确认通知，会回查本地事务，查看本地事务是否执⾏成功，假如本地事务执⾏成功，那么会发送消息，假如本地事务执⾏失败，那么会丢弃事务，假如本地事务还在初始态，那么会过⼀会再来询问
5. 消息在MQ订阅⽅的消息消费成功由MQ来保证



**如何保证？（重要）**

- MQ假如投递消息到MQ订阅⽅失败了，或者MQ订阅⽅消费消息失败了，那么MQ会把该消息丢⼊重试队列中，会重试发送该消息，默认16次，直到消息被消费成功为⽌
- 假如在16次之后该消息还没有被消费成功，那么MQ会再次把该消息丢⼊MQ死信队列中，对于死信队列的消息，我们需要⼿动去⼲预，让他消费成功（例如从后台管理系统⼿动（或者是定时任务）把死信队列中的消息拿出来，然后⼿动去执⾏操作，执⾏完成之后把消息从死信队列中删除掉）



#### 3.3.3 其他解决⽅案

**阿⾥GTS**
成熟的⽅案，⼀站式解决，需要付费
参考链接：https://helpcdn.aliyun.com/product/48444.html

**基于GTS的免费社区版本，SEATA**
参考链接：https://github.com/seata/seata



## 4 消息型事务代码实现



# 限流

## 1 概念介绍

⾼并发相关的概念：

- PV：综合浏览量（PageView），即⻚⾯浏览量或者点击量，如果⼀个系统的⽇PV在千万级以上，那么我们称这个系统可能为⾼并发系统
- QPS：每秒钟请求或者查询的数量，在互联⽹领域，指每秒响应请求数（指HTTP请求）。
- 响应时间：从请求发出到收到响应花费的时间。例如系统处理⼀个 HTTP请求需要100ms，这个100ms就是系统的响应时间。
- 吞吐量：单位时间内处理的请求数量。



如何实现⼀个系统的⾼并发呢？或者说如何提⾼⼀个系统的并发量呢？

1. 扩容：
由于单台服务器的处理能⼒有限，因此当⼀台服务器的处理能⼒接近或已超出其容量上限时，采⽤集群技术对服务器进⾏扩容，可以很好地提升系统整体的并⾏处理能⼒，在集群环境中，节点的数量越多，系统的并⾏能⼒和容错性就越强。
在<mark>⽆状态服务</mark>下，扩容可能是迄今为⽌效果最明显的增加并发量的技巧之⼀。
从扩容⽅式⻆度讲，分为垂直扩容（scale up）和⽔平扩容（scale out）。垂直扩容就是增加单机处理能⼒，怼硬件，但硬件能⼒毕竟还是有限；⽔平扩容说⽩了就是增加机器数量，怼机器，但随着机器数量的增加，单应⽤并发能⼒并不⼀定与其呈现线性关系， 此时就可能需要进⾏应⽤服务化拆分了。
2. 提⾼接⼝并发能⼒
- 缓存
- 动静分离
  动静分离是指，静态⻚⾯与动态⻚⾯分开不同系统访问的架构设计⽅法。
  ⼀般来说：
  - 静态⻚⾯访问路径短，访问速度快，⼏毫秒
  - 动态⻚⾯访问路径⻓，访问速度相对较慢(数据库的访问，⽹络传输，业务逻辑计算)，
    ⼏⼗毫秒甚⾄⼏百毫秒，对架构扩展性的要求更⾼
- 服务降级
  业务⾼峰期，为了保证核⼼服务，需要停掉⼀些不太重要的业务。
- 限流
  顾名思义，限制系统的流量
  做法：通过对并发访问和请求进⾏限速或者⼀个时间窗⼝内的请求进⾏限速来保护系统的可⽤性，⼀旦达到限制速率就可以拒绝服务（友好定向到错误⻚或告知资源没有了），排队或者等待（⽐如秒杀，评论，下单）



⾼并发三把利器：

- 缓存：（提⾼并发量）
  提升系统访问速度，增⼤系统处理流量
- 限流：限制流量，保护系统（提⾼可⽤性）
  假如 500个请求/s 1000个请求/s 等待操作 如果我们⽤了限流策略之后，我们只需要让500个请求去访问我们的系统就好。
  限流的⽬的是通过对并发访问/请求进⾏限速，或者对⼀个时间窗⼝内的请求进⾏限速来保护系统，⼀旦达到限制速率则可以拒绝服务、排队或等待、降级等处理
- 降级：（提⾼可⽤性）
  降级是当服务器压⼒剧增的情况下，根据当前业务情况及流量对⼀些服务和⻚⾯有策略的降级，以此释放服务器资源以保证核⼼任务的正常运⾏



## 2 库存售罄问题优化

问题：如果库存已经售罄，那么后续的⼤流量会对数据库产⽣不必要的压⼒，这⾥可以使⽤Redis在库存售罄之后打上⼀个库存售罄的标记，我们可以通过这个标记再去判断是否可以进⾏秒杀下单



## 3 流量削峰技术

流量削峰技术的由来：
主要还是来⾃于互联⽹秒杀抢购等业务场景。例如阿⾥双⼗⼀秒杀，短时间上亿⽤户的涌⼊，瞬间流量巨⼤，⽐如100万⼈同事抢购⼀个商品，⽽商品的库存是100件，这样真实能购买到商品的⼈数也就是在100⼈。从业务上来讲，秒杀活动是希望有很多的⼈参与进来，也就是抢购之前有更多的⼈看到商品。但是，在抢购时间到达之后，⽤户开始真正下单时，我们秒杀服务器的后端是绝对不希望有100w⽤户来同时下单，发起抢购请求。

因为我们都知道服务器处理资源是有限的，所以在出现流量峰值时，很容易导致服务器宕机，出现⽤户⽆法访问的情况



所以我们需要降低流量的峰值！！！
那么如何降低呢？就好⽐政府为了解决早晚⾼峰出⾏的问题，于是就有了错峰限⾏的解决⽅案，本质上其实就是限制⼀部分流量
那么在互联⽹抢购业务场景之中，我们就有了流量削峰技术。
解决⽅案⼤致有以下⼏种办法



### 3.1 验证码技术

### 3.2 限流算法

#### 3.2.1 令牌桶算法

![image-20210628093515693](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210628093515693.png)



#### 3.2.2 漏桶算法

![image-20210628140720204](C:\Users\AnoxiC2010\Documents\GitHub\Hello-World\Java\Microservice-project-notes.assets\image-20210628140720204.png)









#### 3.2.3 优缺点⽐较

- 漏桶算法：能否强⾏限制请求处理的速度
- 令牌桶算法：除了能够在总体上限制数据的平均传输速率以外，还允许某种程度上的突发传输。在令牌桶的传输中，只要令牌桶中存在令牌，那么就允许突发的数据传输直到⽤户配置的上限，因此令牌桶更加适⽤于突发流量。



#### 3.2.4 令牌桶实例

Google提供 的Guava⼯具包中的RateLimiter进⾏限流



### 3.3 队列泄洪

原理：
本质：排队策略
排队有时候⽐并发更加⾼效（例如Redis单线程模型，innodb的mutex key等）
Alisql对innodb的优化策略（让请求去排队 排队去处理sql请求）
依靠排队去限制并发流量
依靠排队和下游拥塞窗⼝的程度调整队列释放流量的⼤⼩



举例：⽀付宝银⾏⽹关队列举例 Zqueue

Zqueue：就是⽀付宝和银⾏对接的⼀个队列

- ⽬的：保护下游系统涌⼊流量，提⾼下游系统的可⽤性
- 代码实现
  线程池











Guava工具包，

dubbo底层使用curator客户端和zookeeper进行通信，在curator的依赖中已经引入了guava的依赖，不需要重复导入



# slf4j日志



# swagger文档



# NIO

传统io是阻塞io  ; new io多路复用模型效率更高



# 能优化的部分



## 密码散列

通知+注解 比 在各方法体中分别使用散列工具类风格和功能更好。

Md5有各种静态工具类，但是在每个方法分别使用工具类，不如配置 通知+注解 ，这样各个方法体只专注于自己的业务即可。

```
 /**
     * 用户激活
     */
    @Anoymous
    @GetMapping("verify")
    public ResponseData verify(String uid, String username) {
        UserVerifyRequest userVerifyRequest = new UserVerifyRequest();
        userVerifyRequest.setUserName(username);
        userVerifyRequest.setUuid(uid);
        UserVerifyResponse response = iVerifyService.userVerify(userVerifyRequest);

        if (response.getCode().equals(SysRetCodeConstants.SUCCESS.getCode())) {
            return new ResponseUtil<>().setData(null);
        }
        return new ResponseUtil().setErrorMsg(response.getCode());
    }
```



## 热点商品信息是用redis

## 秒杀延迟mq返还秒杀库存

# BUG

## 网关层启动不了，没使用数据源，却提示没配置DataSource url属性

问题:

网关层一开始没问题，后来某次git pull后启动不了，错误如下。找不到原因，毕竟网关层并没有使用到数据源。

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class


Action:

Consider the following:
	If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
	If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).
```



解决：

项目结构：

```
分布式微服务 dubbo sprigboot Project下分不同module
gateway #都是controller
mall-commons
	commons-core
	commons-lock
	commons-mq
	commons-tool
mall-parent
order-service
	order-api
	order-provider
shopping-service
	shopping-api
	shopping-provider
user-service
	user-api
	user-provider
	user-sdk
```

由于网关层不需要依赖数据源，导入的依赖一般来自

mall-parent、mall-commons中的、各个service模块层的api

查遍所有pom.xml之后在队友已经无耐先给网关配置数据源用于启动之后，回头在查看网关层的pom.xml发现引入了一个provider层，就是这个provider层的依赖激活了网关层的数据源自动配置要求。

注释掉之后正常启动。



## 网关层不能debug启动，只能run

断点问题，取消断点





## org.apache.rocketmq.client.exception.MQClientException: No name server address, please set it.

神奇的bug

代码里setNamesrvAddr("127.0.0.1:9876")是写死的

订单模块和秒杀模块的producer都没有问题

用户模块报了这个错误

```java
@Test
    public void testMqError() throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        

        DefaultMQProducer producer = new DefaultMQProducer("test_send_register");

        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();

//        try {
//            System.in.read();
//        } catch (IOException e) {
//            e.printStackTrace();
//        }
        Message test_register = new Message("test_register", "aliang".getBytes());

        SendResult send = producer.send(test_register);
        System.out.println(send);
    }
```

在springboot的单元测试中也报了这个错误。

邪门的是把这个测试方法改为main方法执行就能成功。




# 其他



spring配置注入

或spi动态发现



核心 Handler方法的添加和执行



事务失效的几种情况

https://zhuanlan.zhihu.com/p/145669970



接口的幂等性

[(7 封私信 / 35 条消息) 分布式高并发系统如何保证对外接口的幂等性？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/27744795/answer/1804031019)





@Service dubbo的这个注解标记的类，已经注入spring容器了，如果不是rpc调用的话，在本地（本module）除了使用@Reference注解还可以直接使用@Autowired来注入使用



项目编码风格造成的事务问题：

本项目的代码风格极为模板化，在service层try...catch了所有异常，异常信息被包裹在了return的模板响应类中。导致了使用springboot的事务极为不便，@Transactional注解是无效的。

如果需要事务环境，要么硬编码回滚事务，要么把事务关联的几个数据库执行语句抽取在一个public方法中标记@Transactional注解然后通过ApplicationContextAware接口获取applicationContex，从applicationContex中获取本类的spring容器代理对象引用并使用，直接在本类中调用是无效的。



# 想法

管道-链表--Java引用-数据结构



