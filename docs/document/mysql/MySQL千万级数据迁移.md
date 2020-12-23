# 千万级数据迁移

## 背景介绍

- Face++ 保存加密的图片和feature混在一起，为了提供服务的质量和高可用性，支持加密服务器的双写和双读，因此需要将OSS中存储图片迁移到一个新的OSS中
- MySQL存储了7000w+的数据
- 需要根据MySQL表中具体某个Json字段中的image_id在OSS中找到对应图片进行迁移

## 思路

#### 第一版简单版：

> 数据量小(14w)

第一版为最基础的版本，第一版仅仅能迁移faceset表中与face表关联的数据。简单可用。

**思路**:

```
SELECT id, anchor FROM face WHERE id in face2set;
从anchor中json解析出image_id
使用set去重
开启30个线程从bucket迁移
```

[python 多线程编程](https://dreamgoing.github.io/Python-threading.md)

 

#### 第二版完整版

> 数据量大(7000w+)

第一版不足：

- 从数据库读数据和去同步数据耦合
- 从数据库读数据不能简单的使用`SELECT id FROM face`这种形式
- 对于异常情况没有做处理

思路：

1. 从数据库读数据和同步数据分离：即分成两个脚本，一边做dump数据库数据，一边做数据同步
2. 去重和支持增量的数据同步
3. 异常情况的处理

## 几个问题

 

#### 问题1: 如何MySQL数据库dump千万级别表数据？

- 热备：不需要任何服务停机时间
- 在备份方法中，最大的问题就是会使用`FLUSH TABLES WITH READ LOCK`操作。
- 锁的开销和锁的粒度之间负相关
- 读锁（read lock）：即共享锁(shared lock)，多个客户可以同一时间同时读取同一个资源
- 写锁 (write lock): 即排他锁(exclusive lock), 即写锁会阻塞其他的写锁和读锁
- 表锁 (table lock): 是MySQL最基本的锁策略且开销最小，用户对表进行更新时使用写锁，读取是使用读锁, `ALTER TABLE` `mysqldump`等操作
- 在特定的场景下`READ LOCAL`表锁, 支持某些类型的并发操作。
- 写锁比读锁有更高的优先级，写锁可以插入到锁队列读锁之前
- 行级锁(row lock): 支持最大程度的并发，开销最大，行级锁只在存储引擎实现，服务层不会实现行级锁。

**mysqldump内部机制**

> mysqldump -uroot -p midas_dev account > midas_dev.sql

![img](https://dreamgoing.github.io/image/mysqldump-midas.png)

- `LOCK TABLES account READ`: 对于account表施加读锁
- `show create table account`: 查看并导出account表的建表语句
- `SELECT * FROM account`: 导出所有的数据
- `SHOW TRIGGERS LIKE account`: 导出trgger
- `UNLOCK TABLES` : 释放对account表的读锁

**SELECT**：

按照偏移量查询，这样可以有针对性的获取线上数据库的数据，避免了使用`SELECT * FROM table`和锁表，控制好每次读取的LIMIT即可以保证对线上数据库几乎无影响。

**结论：**

- `mysqldump`: 只适合用于数据库备份，不适合于从数据库取数据，即使是在数据库业务低峰期，也会对线上业务的有一定的影响。
- `SELECT`: 适合于从线上数据库读取数据，不适合做数据库的备份，即不能保证数据库的一致性，对业务的影响低，适合于当前数据迁移的场景。

#### 问题2: `OFFSET LIMIT` vs `BETWEEN AND`？

**OFFSET LIMIT**😪

执行如下语句N从1000到1000000.

```
SELECT ID, anchor FROM face LIMIT 1000 OFFSET N;
```

| N        | 耗时   |
| -------- | ------ |
| 1000     | 192ms  |
| 10000    | 653ms  |
| 100000   | 985ms  |
| 1000000  | 1s4ms  |
| 10000000 | 5s25ms |

由上述耗时可以看出`LIMIT OFFSET`语句的查询效率受限于OFFSET N。

原因很简单, 即`LIMIT 200 OFFSET 1000`: 是扫描数据表前1200条记录，取后200条舍弃前1000条。

**BETWEEN AND👊**

执行如下语句获取N条数据.

```
SELECT id, anchor FROM face WHERE id BETWEEN A AND B;
```

| N      | 耗时    |
| ------ | ------- |
| 853    | 142ms   |
| 45149  | 1s300ms |
| 62835  | 1s630ms |
| 646765 | 1s543ms |
|        |         |

上述耗时可以看出在不同的A，B区间内获取不同数量的内容，耗时均在2s以内，与A，B位置无关。

原因即，`BETWEEN A AND B`利用MySQL主键索引的特定，没有额外的数据查询，即这样可以在`O(logn)`的时间复制度内完成查询。

![img](https://dreamgoing.github.io/image/innodb-btree.png)

上图是MySQL存储引擎Innodb，B+数索引的结构，即可以看到B+数叶子节点会索引对应的文件，当进行范围查找时，通过查找A第一个主键找到叶子节点的开始节点文件指针，再查找B找到结束节点的叶子指针，根据B Tree的特定，以及磁盘顺序IO，即可以按照磁盘顺序IO读取所有要的范围的数据。查询复杂度为O(logn).

#### 问题3: 千万级别的数据如何去重？

> hash, set, trie, bloomfilter, external sorting ...

数据量：7000w*30byte = 2.1G， 该数据量可以存储在内存中 4c8g。

**hash**: O(n)

- golang map 80s
- c++ unordered_set 72s
- python set 120s

**set**: O(nlogn)

- c++ set 250s (红黑树)

**trie**: O(BASE64_SIZE*key_length*N)

- c++ trie>250s

#### 问题4: 当同步数据的脚本奔溃了，怎么断点续传？

整个同步的脚本是分离的，即一个脚本为数据库同步，直接根据上一次同步的最大自增主键继续进行查找，另一个脚本为图片传输同步。

图片传输过程中实时记录，已成功的图片id，和发生错误的图片id，在程序崩溃之后可以根据已保存的id和全部id获得未保存的id，并将发生错误的图片id重新同步。

#### 问题5: `Python` 多线程编程的范式？

> 这里我只提供个人觉得很好用的一个范式，多生产者单消费者模型，这种多对一，或者一对多的情况可以使用无锁化的队列实现。

- imageid_queue, save_queue, error_queue 三个线程安全的队列
- `sync_producer`: 读取图片id的生产者，将图片id放入`imageid_queue`中。（一个生产者）
- `sync_worker`:同步图片worker，从imageid_queue中获取图片id，并执行迁移操作。（多个消费者）
- save_worker:保存已成功迁移图片的worker，从save_queue中获取图片id，并写入文件。（与文件操作相关，单消费者保证了对文件写的线程安全）
- `error_worker`:保存迁移图片失败的worker，从`error_queue`中获取图片id，并写入文件。（同上）
- 线程之间的同步使用`Event`, 线程之间的资源控制使用`Mutex`

## 源码

> Talk is cheap, show me your code

从MySQL数据库同步数据代码简单，单线程，增量同步即可。介绍数据同步脚本

![img](https://dreamgoing.github.io/image/sync_arch.jpg)

 

```
# coding: utf-8
import csv
import functools
import time
import threading
import Queue
import base64

import oss2

ENV = "PROD"  # NOTE PROD or TEST(同步很一小部分图片到测试10.104.4.50:12029进行验证)

BJ_OSS_ENDPOINT = 'oss-cn-beijing.aliyuncs.com'  # 'oss-cn-beijing-internal.aliyuncs.com'

NEW_BUCKET = {
    'access_id': '*******',
    'access_key': '*******',
    'bucket_name': '*******'
}

OLD_BUCKET = {
    'origin': '*******',
    'access_id': '*******',
    'access_key': '*******',
    'bucket_name': '*******'
}

TEST_BUCKET = {
    'origin': '*******',
    'access_id': '*******',
    'access_key': '*******',
    'bucket_name': '*******'
}

# MySqlDB
HOST = '*******'
USER = '*******'
PASSWORD = '*******'
DB = '*******'

# file cache
LATEST_IMAGES_PATH = "./mysql_image.csv"
SAVED_IMAGES_PATH = "./saved_image.csv"
ERROR_IMAGES_PATH = './error_image.csv'

new_auth = oss2.Auth(NEW_BUCKET['access_id'], NEW_BUCKET['access_key'])
new_bucket = oss2.Bucket(new_auth, BJ_OSS_ENDPOINT, NEW_BUCKET['bucket_name'], connect_timeout=5)

old_auth = oss2.Auth(OLD_BUCKET['access_id'], OLD_BUCKET['access_key'])
old_bucket = oss2.Bucket(old_auth, BJ_OSS_ENDPOINT, OLD_BUCKET['bucket_name'])

test_auth = oss2.Auth(TEST_BUCKET['access_id'], TEST_BUCKET['access_key'])
test_bucket = oss2.Bucket(test_auth, BJ_OSS_ENDPOINT, TEST_BUCKET['bucket_name'])

MAX_QUEUE_SIZE = 100000
imageid_queue = Queue.Queue(maxsize=MAX_QUEUE_SIZE)
check_queue = Queue.Queue(maxsize=MAX_QUEUE_SIZE)
error_queue = Queue.Queue(maxsize=MAX_QUEUE_SIZE)
save_queue = Queue.Queue(maxsize=MAX_QUEUE_SIZE)

end_event = threading.Event()

WORKER_NUM = 30
CHECK_WORKER_NUM = 30

LOG_LIMIT = 1000

# 总数 2018-11-12 15:27 数量
TOTAL = 74110364


class Counter(object):
    def __init__(self, start=0):
        self.lock = threading.Lock()
        self.value = start

    def increment(self):
        current_threading = threading.currentThread()

        self.lock.acquire()
        try:
            self.value = self.value + 1
            print "{}: worker finish. counter: {}".format(current_threading, self.value)

            if self.value == WORKER_NUM:
                print "all worker finish"
                end_event.set()

        finally:
            self.lock.release()


counter = Counter()


def omit_exception(method=None, return_value=None):
    if method is None:
        return functools.partial(omit_exception, return_value=return_value)

    @functools.wraps(method)
    def _decorator(*args, **kwargs):
        try:
            return method(*args, **kwargs)
        except Exception as e:
            print "{}: {}".format(method.__name__, e)
            return return_value

    return _decorator


def timeit(method):
    @functools.wraps(method)
    def _decorator(*args, **kwargs):
        ts = time.time()
        result = method(*args, **kwargs)
        te = time.time()
        print ("[function] {} run: {}ms {}min".format(method.__name__, round((te - ts) * 1000), round((te - ts) / 60)))
        return result

    return _decorator


@timeit
def get_update_image():
    latest_image = set()
    saved_image = set()
    cnt = 0
    with open(LATEST_IMAGES_PATH, 'r') as f:
        csv_reader = csv.reader(f, delimiter=',')
        for it in csv_reader:
            if cnt % LOG_LIMIT == 0:
                print cnt
            cnt += 1
            image_id = base64.urlsafe_b64encode(base64.b64decode(it[1]))
            latest_image.add(image_id)

    with open(SAVED_IMAGES_PATH, 'r') as f:
        csv_reader = csv.reader(f, delimiter=',')

        for it in csv_reader:
            saved_image.add(it[0])

    update_image = latest_image - saved_image
    print ("need sync: {} saved: {}".format(len(update_image), len(saved_image)))
    return update_image


@timeit
def get_save_image():
    saved_image = set()
    with open(SAVED_IMAGES_PATH, 'r') as f:
        csv_reader = csv.reader(f, delimiter=',')

        for it in csv_reader:
            saved_image.add(it[0])
    return saved_image


@timeit
def sync_producer():
    update_image = get_update_image()
    cnt = 1

    for it in update_image:
        imageid_queue.put((cnt, it))
        cnt = cnt + 1

    for _ in range(WORKER_NUM):
        imageid_queue.put((cnt, "end"))

    while not end_event.isSet():
        event_is_set = end_event.wait()

        if event_is_set:
            error_queue.put("end")
            save_queue.put("end")


def sync_worker(total=TOTAL):
    st = time.time()
    current_threading = threading.currentThread()
    print "{} worker start".format(current_threading)
    while True:
        idx, image_id = imageid_queue.get()
        if image_id == "end":
            break

        now = time.time()
        time_used = round(now - st, 2)
        content = get_content(image_id)

        success = True
        if content is None:
            print "Fs Oss Error: image_id: {}".format(image_id)
            success = False
        oss_resp = put_oss(image_id, content)
        if oss_resp is None or oss_resp.resp.status != 200:
            print "Oss Error: image_id: {}".format(image_id)
            success = False

        if success:
            save_queue.put(image_id)
        else:
            error_queue.put(image_id)

        if idx % 1000 == 0:
            print "{}/{} {} {}s 还需要: {}min".format(idx, total, image_id, time_used, round((time_used * total) / (
                    idx * 60) - time_used / 60, 2))

    counter.increment()


def error_worker():
    with open(ERROR_IMAGES_PATH, 'a') as f:
        csv_writer = csv.writer(f, delimiter=',')
        while True:
            image_id = error_queue.get()
            if image_id == "end":
                break
            csv_writer.writerow([image_id])


def save_worker():
    with open(SAVED_IMAGES_PATH, 'a') as f:
        csv_writer = csv.writer(f, delimiter=',')
        while True:
            image_id = save_queue.get()
            if image_id == "end":
                break
            csv_writer.writerow([image_id])


@omit_exception()
def get_content(image_id):
    res = old_bucket.get_object(image_id)
    if res is not None and res.resp.status == 200:
        content = res.read()
        return content


@omit_exception()
def put_oss(key, content):
    if ENV == "TEST":
        res = test_bucket.put_object(key, content)
    elif ENV == "PROD":
        res = new_bucket.put_object(key, content)
    else:
        print "ENV error!!!!"
    return res


def get_oss(key):
    res = new_bucket.get_object("hello")
    print res.resp.status, res.read()


@timeit
def run_sync():
    total = 34595436
    read_t = threading.Thread(target=sync_producer)
    read_t.start()

    error_t = threading.Thread(target=error_worker)
    error_t.start()

    save_t = threading.Thread(target=save_worker)
    save_t.start()

    for i in range(WORKER_NUM):
        worker_t = threading.Thread(target=sync_worker, args=(total,))
        worker_t.start()

    main_thread = threading.currentThread()

    for t in threading.enumerate():
        if t is not main_thread:
            t.join()

    print "sync successfully!"


def remove_duplicate():
    image_set = set()
    with open(SAVED_IMAGES_PATH, 'rb') as f:
        csv_reader = csv.reader(f, delimiter=',')
        for it in csv_reader:
            image_set.add(it[0])

    with open(SAVED_IMAGES_PATH, 'wb') as f:
        csv_writer = csv.writer(f, delimiter=',')
        for it in image_set:
            csv_writer.writerow([it])


if __name__ == '__main__':
    run_sync()
    # remove_duplicate()

```









## 记录一次MySQL两千万数据的大表优化解决过程，提供三种解决方案

使用阿里云rds for MySQL数据库(就是MySQL5.6版本)，有个用户上网记录表6个月的数据量近2000万，保留最近一年的数据量达到4000万，查询速度极慢，日常卡死。严重影响业务。我尝试解决该问题，so，有个这个日志。



### 问题概述

使用阿里云rds for MySQL数据库(就是MySQL5.6版本)，有个用户上网记录表6个月的数据量近2000万，保留最近一年的数据量达到4000万，查询速度极慢，日常卡死。严重影响业务。

问题前提：老系统，当时设计系统的人大概是大学没毕业，表设计和sql语句写的不仅仅是垃圾，简直无法直视。原开发人员都已离职，到我来维护，这就是传说中的维护不了就跑路，然后我就是掉坑的那个!!!

我尝试解决该问题，so，有个这个日志。

### 方案概述

- 方案一：优化现有mysql数据库。优点：不影响现有业务，源程序不需要修改代码，成本***。缺点：有优化瓶颈，数据量过亿就玩完了。
- 方案二：升级数据库类型，换一种100%兼容mysql的数据库。优点：不影响现有业务，源程序不需要修改代码，你几乎不需要做任何操作就能提升数据库性能，缺点：多花钱
- 方案三：一步到位，大数据解决方案，更换newsql/nosql数据库。优点：扩展性强，成本低，没有数据容量瓶颈，缺点：需要修改源程序代码

以上三种方案，按顺序使用即可，数据量在亿级别一下的没必要换nosql，开发成本太高。三种方案我都试了一遍，而且都形成了落地解决方案。该过程心中慰问跑路的那几个开发者一万遍 :)

### 方案一详细说明：优化现有mysql数据库

跟阿里云数据库大佬电话沟通 and Google解决方案 and 问群里大佬，总结如下(都是精华)：

- 1.数据库设计和表创建时就要考虑性能
- 2.sql的编写需要注意优化
- 3.分区
- 4.分表
- 5.分库

**1、数据库设计和表创建时就要考虑性能**

mysql数据库本身高度灵活，造成性能不足，严重依赖开发人员能力。也就是说开发人员能力高，则mysql性能高。这也是很多关系型数据库的通病，所以公司的dba通常工资巨高。

设计表时要注意：

1. 表字段避免null值出现，null值很难查询优化且占用额外的索引空间，推荐默认数字0代替null。
2. 尽量使用INT而非BIGINT，如果非负则加上UNSIGNED(这样数值容量会扩大一倍)，当然能使用TINYINT、SMALLINT、MEDIUM_INT更好。
3. 使用枚举或整数代替字符串类型
4. 尽量使用TIMESTAMP而非DATETIME
5. 单表不要有太多字段，建议在20以内
6. 用整型来存IP

**索引**

1. 索引并不是越多越好，要根据查询有针对性的创建，考虑在WHERE和ORDER BY命令上涉及的列建立索引，可根据EXPLAIN来查看是否用了索引还是全表扫描
2. 应尽量避免在WHERE子句中对字段进行NULL值判断，否则将导致引擎放弃使用索引而进行全表扫描
3. 值分布很***的字段不适合建索引，例如"性别"这种只有两三个值的字段
4. 字符字段只建前缀索引
5. 字符字段***不要做主键
6. 不用外键，由程序保证约束
7. 尽量不用UNIQUE，由程序保证约束
8. 使用多列索引时主意顺序和查询条件保持一致，同时删除不必要的单列索引

**简言之就是使用合适的数据类型，选择合适的索引**

选择合适的数据类型 (1)使用可存下数据的最小的数据类型，整型 < date,time < char,varchar < blob (2)使用简单的数据类型，整型比字符处理开销更小，因为字符串的比较更复杂。如，int类型存储时间类型，bigint类型转ip函数 (3)使用合理的字段属性长度，固定长度的表会更快。使用enum、char而不是varchar (4)尽可能使用not null定义字段 (5)尽量少用text，非用不可***分表 # 选择合适的索引列 (1)查询频繁的列，在where，group by，order by，on从句中出现的列 (2)where条件中<，<=，=，>，>=，between，in，以及like 字符串+通配符(%)出现的列 (3)长度小的列，索引字段越小越好，因为数据库的存储单位是页，一页中能存下的数据越多越好 (4)离散度大(不同的值多)的列，放在联合索引前面。查看离散度，通过统计不同的列值来实现，count越大，离散程度越高：

原开发人员已经跑路，该表早已建立，我无法修改，故：该措辞无法执行，放弃!

### 2、sql的编写需要注意优化

1. 使用limit对查询结果的记录进行限定
2. 避免select *，将需要查找的字段列出来
3. 使用连接(join)来代替子查询
4. 拆分大的delete或insert语句
5. 可通过开启慢查询日志来找出较慢的SQL
6. 不做列运算：SELECT id WHERE age + 1 = 10，任何对列的操作都将导致表扫描，它包括数据库教程函数、计算表达式等等，查询时要尽可能将操作移至等号右边
7. sql语句尽可能简单：一条sql只能在一个cpu运算;大语句拆小语句，减少锁时间;一条大sql可以堵死整个库
8. OR改写成IN：OR的效率是n级别，IN的效率是log(n)级别，in的个数建议控制在200以内
9. 不用函数和触发器，在应用程序实现
10. 避免%xxx式查询
11. 少用JOIN
12. 使用同类型进行比较，比如用'123'和'123'比，123和123比
13. 尽量避免在WHERE子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描
14. 对于连续数值，使用BETWEEN不用IN：SELECT id FROM t WHERE num BETWEEN 1 AND 5
15. 列表数据不要拿全表，要使用LIMIT来分页，每页数量也不要太大

原开发人员已经跑路，程序已经完成上线，我无法修改sql，故：该措辞无法执行，放弃!

### 引擎

引擎

目前广泛使用的是MyISAM和InnoDB两种引擎：

MyISAM

MyISAM引擎是MySQL 5.1及之前版本的默认引擎，它的特点是：

1. 不支持行锁，读取时对需要读到的所有表加锁，写入时则对表加排它锁
2. 不支持事务
3. 不支持外键
4. 不支持崩溃后的安全恢复
5. 在表有读取查询的同时，支持往表中插入新纪录
6. 支持BLOB和TEXT的前500个字符索引，支持全文索引
7. 支持延迟更新索引，极大提升写入性能
8. 对于不会进行修改的表，支持压缩表，极大减少磁盘空间占用

InnoDB

InnoDB在MySQL 5.5后成为默认索引，它的特点是：

1.支持行锁，采用MVCC来支持高并发

2.支持事务

3.支持外键

4.支持崩溃后的安全恢复

5.不支持全文索引

**总体来讲，MyISAM适合SELECT密集型的表，而InnoDB适合INSERT和UPDATE密集型的表**

MyISAM速度可能超快，占用存储空间也小，但是程序要求事务支持，故InnoDB是必须的，故该方案无法执行，放弃!

### **3、分区**

MySQL在5.1版引入的分区是一种简单的水平拆分，用户需要在建表的时候加上分区参数，对应用是透明的无需修改代码

对用户来说，分区表是一个独立的逻辑表，但是底层由多个物理子表组成，实现分区的代码实际上是通过对一组底层表的对象封装，但对SQL层来说是一个完全封装底层的黑盒子。MySQL实现分区的方式也意味着索引也是按照分区的子表定义，没有全局索引

用户的SQL语句是需要针对分区表做优化，SQL条件中要带上分区条件的列，从而使查询定位到少量的分区上，否则就会扫描全部分区，可以通过EXPLAIN PARTITIONS来查看某条SQL语句会落在那些分区上，从而进行SQL优化，我测试，查询时不带分区条件的列，也会提高速度，故该措施值得一试。

**分区的好处是：**

1. 可以让单表存储更多的数据
2. 分区表的数据更容易维护，可以通过清楚整个分区批量删除大量数据，也可以增加新的分区来支持新插入的数据。另外，还可以对一个独立分区进行优化、检查、修复等操作
3. 部分查询能够从查询条件确定只落在少数分区上，速度会很快
4. 分区表的数据还可以分布在不同的物理设备上，从而搞笑利用多个硬件设备
5. 可以使用分区表赖避免某些特殊瓶颈，例如InnoDB单个索引的互斥访问、ext3文件系统的inode锁竞争
6. 可以备份和恢复单个分区

**分区的限制和缺点：**

1. 一个表最多只能有1024个分区
2. 如果分区字段中有主键或者唯一索引的列，那么所有主键列和唯一索引列都必须包含进来
3. 分区表无法使用外键约束
4. NULL值会使分区过滤无效
5. 所有分区必须使用相同的存储引擎

**分区的类型：**

1. RANGE分区：基于属于一个给定连续区间的列值，把多行分配给分区
2. LIST分区：类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择
3. HASH分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL中有效的、产生非负整数值的任何表达式
4. KEY分区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值
5. 具体关于mysql分区的概念请自行google或查询官方文档，我这里只是抛砖引玉了。

我首先根据月份把上网记录表RANGE分区了12份，查询效率提高6倍左右，效果不明显，故：换id为HASH分区，分了64个分区，查询速度提升显著。问题解决!

结果如下：

```
PARTITION BY HASH (id)PARTITIONS 64 
select count() from readroom_website; --11901336行记录  / 受影响行数: 0 已找到记录: 1 警告: 0 持续时间 1 查询: 5.734 sec. /  select * from readroom_website where month(accesstime) =11 limit 10;  / 受影响行数: 0 已找到记录: 10 警告: 0 持续时间 1 查询: 0.719 sec. */ 
```

### 4、分表

分表就是把一张大表，按照如上过程都优化了，还是查询卡死，那就把这个表分成多张表，把一次查询分成多次查询，然后把结果组合返回给用户。

分表分为垂直拆分和水平拆分，通常以某个字段做拆分项。比如以id字段拆分为100张表： 表名为 tableName_id%100

但：分表需要修改源程序代码，会给开发带来大量工作，极大的增加了开发成本，故：只适合在开发初期就考虑到了大量数据存在，做好了分表处理，不适合应用上线了再做修改，成本太高!!!而且选择这个方案，都不如选择我提供的第二第三个方案的成本低!故不建议采用。

### 5、分库

把一个数据库分成多个，建议做个读写分离就行了，真正的做分库也会带来大量的开发成本，得不偿失!不推荐使用。

### 方案二详细说明：升级数据库，换一个100%兼容mysql的数据库

mysql性能不行，那就换个。为保证源程序代码不修改，保证现有业务平稳迁移，故需要换一个100%兼容mysql的数据库。

**开源选择**

1. tiDB https://github.com/pingcap/tidb
2. Cubrid https://www.cubrid.org/
3. 开源数据库会带来大量的运维成本且其工业品质和MySQL尚有差距，有很多坑要踩，如果你公司要求必须自建数据库，那么选择该类型产品。

云数据选择

1. 阿里云POLARDB
2. https://www.aliyun.com/product/polardb?spm=a2c4g.11174283.cloudEssentials.47.7a984b5cS7h4wH

官方介绍语：POLARDB 是阿里云自研的下一代关系型分布式云原生数据库，100%兼容MySQL，存储容量***可达 100T，性能***提升至 MySQL 的 6 倍。POLARDB 既融合了商业数据库稳定、可靠、高性能的特征，又具有开源数据库简单、可扩展、持续迭代的优势，而成本只需商用数据库的 1/10。

我开通测试了一下，支持免费mysql的数据迁移，无操作成本，性能提升在10倍左右，价格跟rds相差不多，是个很好的备选解决方案!

1. 阿里云OcenanBase
2. 淘宝使用的，扛得住双十一，性能***，但是在公测中，我无法尝试，但值得期待
3. 阿里云HybridDB for MySQL (原PetaData)
4. https://www.aliyun.com/product/petadata?spm=a2c4g.11174283.cloudEssentials.54.7a984b5cS7h4wH

官方介绍：云数据库HybridDB for MySQL (原名PetaData)是同时支持海量数据在线事务(OLTP)和在线分析(OLAP)的HTAP(Hybrid Transaction/Analytical Processing)关系型数据库。

我也测试了一下，是一个olap和oltp兼容的解决方案，但是价格太高，每小时高达10块钱，用来做存储太浪费了，适合存储和分析一起用的业务。

1. 腾讯云DCDB
2. https://cloud.tencent.com/product/dcdb_for_tdsql

官方介绍：DCDB又名TDSQL，一种兼容MySQL协议和语法，支持自动水平拆分的高性能分布式数据库——即业务显示为完整的逻辑表，数据却均匀的拆分到多个分片中;每个分片默认采用主备架构，提供灾备、恢复、监控、不停机扩容等全套解决方案，适用于TB或PB级的海量数据场景。

腾讯的我不喜欢用，不多说。原因是出了问题找不到人，线上问题无法解决头疼!但是他价格便宜，适合超小公司，玩玩。

### 方案三详细说明：去掉mysql，换大数据引擎处理数据

数据量过亿了，没得选了，只能上大数据了。

**开源解决方案**

hadoop家族。hbase/hive怼上就是了。但是有很高的运维成本，一般公司是玩不起的，没十万投入是不会有很好的产出的!

**云解决方案**

这个就比较多了，也是一种未来趋势，大数据由专业的公司提供专业的服务，小公司或个人购买服务，大数据就像水/电等公共设施一样，存在于社会的方方面面。

国内做的***的当属阿里云。

我选择了阿里云的MaxCompute配合DataWorks，使用超级舒服，按量付费，成本极低。

MaxCompute可以理解为开源的Hive，提供sql/mapreduce/ai算法/python脚本/shell脚本等方式操作数据，数据以表格的形式展现，以分布式方式存储，采用定时任务和批处理的方式处理数据。DataWorks提供了一种工作流的方式管理你的数据处理任务和调度监控。

当然你也可以选择阿里云hbase等其他产品，我这里主要是离线处理，故选择MaxCompute，基本都是图形界面操作，大概写了300行sql，费用不超过100块钱就解决了数据处理问题。