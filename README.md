# redis-编辑了Redis的使用教程，拿来上手就用
安装软件Typora查看
Redis和tp使用
类型
支持多种数据结构类型，包括string（字符串）、hash(哈希)、list(链表)、set(无序集合)、Zset(有序集合)

Redis能读的速度是110000次/s,写的速度是81000次/s 。

Redis所有单个命令的执行都是原子性的。

适合场合及其优势
（1）高性能缓存，最常见的应用场景

（2）多类型数据结构，适合各种类型数据，

（3）Redis分布式存储

（4）数据有生命周期，Redis的键可以设置过期时长，一段时间以后自动删除。

（5）高并发和海量数据的处理

（6）数据持久化，数据存储到硬盘里面，服务器断电不丢失。

原生先连接
php安装redis

$redis = new Redis(); $redis->connect('127.0.0.1', 6379, 60);//连接redis服务，指定主机，端口，和超时时间

$redis->auth('123456');//密码
切换数据库
//tp
Cache::select(2);
原生
$redis = new Redis();
$redis->connect('127.0.0.1', 6379, 60);//连接redis服务，指定主机，端口，和超时时间
$redis->select(2);
原生
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->auth('123456');//密码
$dade = $redis->set("name", "dade");
echo $redis->get('name');
（字符串）string
string是redis最基本的类型

redis的string可以包含任何数据，包括序列化的对象、数组。

单个value值最大上限是1G字节, 如果只用string类型，redis就可以被看作加上持久化特性（服务器重启之后，数据不丢失）的memcache

应用场景，缓存HTML页面，关系型数据库的查询结果，统计网站访问者数量，每天注册用户数，统计活跃用户，用户在线状态等等。

（1）set保存
语法：set  键名称  值 
返回值，设置成功返回OK，设置失败返回nil
注意：重新设置则直接覆盖。
Cache::set('name', '大得0',36000);//缓存
Cache::get('name');//查
原生
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$dade = $redis->set("name", "dade");
echo $redis->get('name');
（2）mset保存多个
语法：mset key value [key value]

同时设置多个key,如果key存在会覆盖，该命令是原子的，所有的键会同时设置成功或失败，成功返回OK

原生
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$dade = $redis->mset(array('name'=>'666','age'=>19));
echo $redis->get('age');
tp6
Cache::mset(array('name'=>'dade','age'=>19));//缓存
echo Cache::get('age');
（3）setex保存设置时间
语法：setex key seconds value

给一个键设置为字符串类型，并指定生存时间（单位：秒），该命令是原子的，如果设置失败或指定生存时间失败，会恢复初始状态。

返回值：如果设置成功，返回OK，如果设置失败，返回错误信息。

//缓存数字，缓存文字和字符串（错）
Cache::setex('dades',100,300);
echo Cache::get('dades');
原生
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->setex('keys',100,10);
echo $redis->get('keys');
（4）setnx设置字符串类型
语法：setnx key value

如果key不存在，将其设置为字符串类型，设置成功，如果该键已经存在，则设置失败。

设置成返回1，设置失败，返回0.

原生
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->setnx('ssss','6666');
echo $redis->get('ssss');
（5）get查
语法：get 键名称

（6）mGet查多个
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(array('key1', 'key2', 'key3')); //array('value1', 'value2', 'value3');
$redis->mGet(array('key0', 'key1', 'key5'));//array(`FALSE`, 'value2', `FALSE`);
（7）strlen查字符串长度
语法：strlen key

返回key的字符串长度，

返回值：字符串长度，如果key不存在返回0，如果不是字符串类型，返回错误信息。

Cache::strlen('ddddss1');
原生
$redis->strlen('ddddss1');
（8）getSet设置新值返回旧值
语法:getset key value

原子的给一个key设置新值，并且将旧值返回，

返回值，如果key不是字符串类型，返回一个错误，

应用场景：比如获取计数器并且重置为0.

$redis->set('x', '42'); 
$exValue = $redis->getSet('x', 'lol');  // return '42'
$redis->get('x')'       // return 'lol'
（9）incr 每执行一次+1
对key的值做加加操作，,每执行一次值加1，值类型要是数据类型。

语法：incr key

将key中存储的数字值增一操作，如果key不存在，则key的值会先被初始化为0，然后再执行incr操作，key的值必须是整型。

Cache::incr('age');
（10）incrBy一次加多少
执行加法的命令，可以指定相加的值，

语法：incrby key 相加的值

$redis->incrBy('key1', 10);
(哈希，存储对象)hash
Redis中的hash类型可以看成是具有key和value的容器，该类型非常适合存储对象信息，

mysql中一行的数据，类似于关联数组。

（1）hset保存对象
设置哈希里面的field和vlaue的值。

语法：hset 哈希的名称（键名称） field value

Cache::hset('name','dade','大得');
Cache::hset('name','age',18);
echo Cache::hget('name','age');
（2）hget读单
Cache::hget('name','age');
（3）hMset一次保存多个
一次性设置多个field和value

$redis->delete('user:1');
$redis->hMset('user:1', array('name' => 'Joe', 'salary' => 2000));
$redis->hIncrBy('user:1', 'salary', 100); 
//tp6
Cache::hMset('names',array('dade'=>123,'age'=>223));
（4）hMget一次获得多个
一次性获取 多个field的value

Cache::hMset('names',array('dade'=>123,'age'=>223));
$ss = Cache::hMget('names',array('dade','age'));
print_r($ss);
（5）hGetAll获得所有
获取指定哈希中所有的field和value

语法：hgetall 哈希的名称

Cache::hMset('names',array('dade'=>123,'age'=>22663));
$ss = Cache::hGetAll('names');
print_r($ss);
（链表，队列，最新，列表）list
list类型其实就是一个字符串的双向链表，按照插入顺序排序。可以添加一个元素到链表的头部（左边）或者尾部（右边），一个链表最多可以包含232-1个元素（4294967295，每个链表超过40亿个元素），这使得list既可以用作栈，也可以用作队列。

应用场景，粉丝列表，最新文章，消息队列等。

（1）lpush头部添加
从链表的头部添加一个或多个元素

语法： lpush 链表的名称（键的名称） 元素，操作为原子性操作，如果key不存在，一个空列表会被创建并执行lpush操作。返回执行lpush命令后，列表的长度。

echo Cache::lpush ('names','大得');//头部添加保存
echo Cache::lpush ('names','大得1');//头部添加保存
echo Cache::lpush ('names','大de2');//头部添加保存
（2）lrange查所有
获取链表里面的元素

语法：lragne 链表的名称 开始下标（索引） 结束下标（索引）

注意：如果开始下标是0，结束下标是-1则是返回链表中所有的元素。

注意：链表里面的元素是序号的（从0开始数，从头部开始），类似于索引数组

$dade =  Cache::lrange('names',0,-1);//读链表全部，数组
print_r($dade);
（3）lInsert插入某个之前
将元素插入到链表中某个元素之前或之后，

语法：lInsert（链表的名称）键 before|after 链表中的某个元素 新的元素。

如果命令执行成功，返回插入操作完成之后，列表的长度；如果没有找到链表中的元素，返回-1；如果链表不存在或空链表，返回0。

$dade =  Cache::lInsert('names', 'before', '大得', '6666');
print_r($dade);
（4）lset修改
修改链表中指定下标的元素；

语法：lset 链表名称 下标 新值

$dade =  Cache::lset('names', 1, '你来啊');
print_r($dade);
（5）lIndex返回指定下标值
返回链表中指定下标的元素

语法：lIndex 链表名 下标

$dade =  Cache::lIndex ('names', 1);
print_r($dade);
（6）rpush尾部添加
从链表的尾部添加元素

语法： rpush 链表的名称（键的名称） 元素

$dade =  Cache::rpush ('names', '尾部添加');
print_r($dade);
（7）llen返回列表的长度
返回列表的长度

语法：llen 链表名称

$dade =  Cache::llen ('names');
print_r($dade);
（8）lpop返回链表中头部的元素并删除
返回链表中头部的元素，并删除。

语法：lpop 链表名称

$dade =  Cache::lpop ('names');
print_r($dade);
（9）lrem删除链表中的元素
删除链表中的元素

语法：lrem key count value

根据参数count的值，删除链表中与value相等的元素。

count>0,从表头开始向表尾搜索，删除与value相等的元素，数量为count;

count<0;从表尾开始向表头搜索，删除与value相等的元素，数量为count;

count=0;删除表中所有与value相等的值，

返回值，被删除的元素的数量

$dade =  Cache::lrem ('names','大得');
print_r($dade);
原生
redis->lPush('key1', 'A');
$redis->lPush('key1', 'B');
$redis->lPush('key1', 'C'); 
$redis->lPush('key1', 'A'); 
$redis->lPush('key1', 'A'); 

$redis->lRange('key1', 0, -1); /* array('A', 'A', 'C', 'B', 'A') */
$redis->lRem('key1', 'A', 2); /* 2 */
$redis->lRange('key1', 0, -1); /* array('C', 'B', 'A') */
（10）ltrim保留指定范围的元素
保留指定范围的元素

语法：ltrim 链表的名称 开始下标 结束下标

$dade =  Cache::ltrim ('names',0,10);
print_r($dade);
（11）rpop返回链表尾部的元素并删除
删除并返回链表尾部的元素

$dade =  Cache::rpop ('names');
print_r($dade);
(无序集合)set
redis的set是string类型的无序集合。

set元素最大可以包含(2的32次方-1)(整型最大值)个元素。

关于set集合类型除了基本的添加、删除操作，其他有用的操作还包含集合的取并集(union)，交集(intersection)，差集(difference)。通过这些操作可以很容易的实现sns中的好友推荐功能。

sina公司的好友关注关系就大量使用了set集合类型。

注意：每个集合中的各个元素不能重复。

该类型应用场合：qq好友推荐。

tom朋友圈(与某某是好友)：mary jack xiaoming wang5 wang6

linken朋友圈(与某某是好友)：yuehan daxiong luce wang5 wang6

（1）sadd向集合中添加元素
向集合中添加元素

语法：sadd 集合名（键名） 元素名称

$ss = Cache::sadd('namesaa','大得');
print_r($ss);
（2）smembers获取集合中的元素
获取集合中的元素

语法：smembers 集合名

$ss = Cache::smembers('namesa');
print_r($ss);
（3）sdiff差集，集合1存在，集合2不存在
获取集合中的差集（在集合1中存在，不在集合2中存在的元素）

语法：sdiff 集合1 集合2

$ss = Cache::sdiff('namesa','namesaa');
print_r($ss);
（4）sinter交集，两个集合中都存在
获取交集（在两个集合中都存在的元素）

语法：sinter 集合1 集合2

$ss = Cache::sinter('namesa','namesaa');
print_r($ss);
（5）sunion两个集合合并去掉重复
求并集（两个集合合并后，去掉重复的元素）

语法：sunion 集合1 集合2

$ss = Cache::sunion('namesa','namesaa');
print_r($ss);
（6）scard获取集合中元素的个数
获取集合中元素的个数

语法：scard 集合名称

$ss = Cache::scard('namesa');
print_r($ss);
(有序集合)zset
sorted set是set的一个升级版本，他在set的基础上增加了一个顺序属性(权值)，这一属性在添加修改元素的时候可以指定，每次指定后，zset会自动重新按新的值调整顺序。操作中的key理解为zset的名字。

注意：序号是可以重复的，在添加元素时，如果该元素已经存在，则更新序号的值。

（1）zadd有序集合中添加元素
向有序集合中添加元素。如果该元素存在，则更新其序号。

语法：zadd 集合名 序号 元素

//序号可重复
$ss = Cache::zadd('dade',1,'dade3');
print_r($ss);
（2）zrange读升续排列
(把集合排序后,返回名次[start,stop]的元素

默认是升续排列

Withscores 是把score也打印出来

按序号升序获取有序集合中的内容，

语法：zrange 集合名称 开始下标 结束下标

$ss = Cache::zrange('dade',0,-1);
print_r($ss);
（3）zrevrange读降序获取
按序号降序获取有序集合中的内容。

语法：zrevrange 集合名称 开始下标(索引) 结束下标（索引）

$ss = Cache::zrevrange('dade',0,-1);
print_r($ss);
（4）zremrangebyrank范围删除
zremrangebyrank 删除集合中排名在指定范围的元素（顺序从小到大排序）

语法：zremrangebyrank key min max

$ss = Cache::zremrangebyrank('dade',0,1);//删除集合0到1的元素
print_r($ss);
（5）zcard返回集合中元素的个数
语法：zcard key

返回有序集合中元素的个数

$ss = Cache::zcard('dade');
print_r($ss);
（6）zscore返回给定元素对应的score
语法：zscore key 元素

返回给定元素对应的score，就是给的序号

$ss = Cache::zscore('dade','dade3');
print_r($ss);
Redis 中的事务
Cache::multi();//开启事务
Cache::lpush('dade','dade3');
Cache::lpush('dade','dade3');
Cache::lpush('dade','dade3');
Cache::exec();//提交事务，提交后指定执行

Cache::multi();//开启事务
Cache::lpush('dade','dade3');
Cache::lpush('dade','dade3');
Cache::lpush('dade','dade3');
Cache::discard();//取消事务，不执行

乐观锁（原生）
set ticket 1
set money 100
watch ticket  //开启检测ticket，有变化，当前事务取消
multi
decr ticket
decrby money 100
exec  //提交事务ticket，监听到有变化,事务被取消
Redis主从复制
网站运行，mysql的写入、读取操作的sql语句比例：1:7

mysql为了降低每个服务器负载，可以设置读写分类(有写服务器、有读取服务器)

为了降低每个redis服务器的负载，可以多设置几个，并做主从模式

一个服务器负载“写”(添加、修改、删除)数据，其他服务器负载“读”数据

主服务器数据会“自动”同步给从服务器

Redis支持简单易用的主从复制（master-slave replication）功能，该功能可以让从服务器（slave server）成为主服务器（master server）的精确复制品。

主要作用：

主从备份，防止主服务器宕机；

读写分离，分担主服务器的任务；

任务分离，从服务器分别担任备份工作和计算工作。

注意点：

Redis使用异步复制，

一个主服务器可以有多个从服务器

不仅主服务器可以有从服务器，从服务器也可以有自己的从服务器，

复制功能不能回阻塞主、从服务器

主服务器配置
requirepass 设置的密码

requirepass 123456

配置：**

（1）bind 127.0.0.1改为 #bind 127.0.0.1

（2）protected-mode yes 改为 protected-mode no（关闭只能本地通信）

从服务器配置
（1）通过slaveof指定自己的角色，主服务器的地址和IP

slaveof 主服务器ip 端口号

（2）从服务器只读

从redis2.6开始，从服务器支持只读模式，通过slave-read-only配置项配置，该模式为从服务器的默认模式

（3）指定从服务器连接主服务器的密码

如果主服务器通过requirepass选项设置了密码，为了让从服务器同步操作顺利进行，通过masterauth配置连接主服务器密码。

配置：

#配置连接主服务器ip 端口

slaveof 192.161.1.69 22

#如果主服务器设置了密码，masterauth 密码

masterauth 123456

#默认就开启，从服务器只读模式

slave-read-only yes

查看是否配置正确

在从服务器执行info replication命令查看是否配置正确；

info replication

如果撤销，只需在从服务器里面屏蔽上面的配置即可。

配置注销就可以

#slaveof 主服务器ip 端口号

#masterauth 123456

redis主从复制的缺陷
每次从服务器断开后，无论是主动断开，还是网络故障，再连接主服务器，从服务器都要从主服务器全部dump出来rdb,再aof;即同步的过程都要重新执行一遍，所以要记住如果是多台从服务器时，不要一下子都启动起来

读数据
自己写，如果多台服务器，代码也在多台，做了负载均衡，写放一台服务器上，读本地redis从服务器，买一台都安装独立的redis（从服务器）。当要写入，封装一个方法，写到主服务器。

或者redis都不在本地，在代码中封装成一个数组，作为读分配。写封装一个方法或者采用框架的配置上主服务器
