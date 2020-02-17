# 关于数据库索引的一些问题


<!--more-->

## 一、密集索引和稀疏索引

- 密集索引文件中的每个搜索码值都对应一个索引值
- 稀疏索引文件只为索引码的某些值建立索引项

![密集索引和稀疏索引](https://i.loli.net/2019/12/24/qtGm8r27hMxLDwV.png)

**MySQL 常见的数据库引擎：MyISAM 只使用稀疏索引，InnoDB 有且仅有一个密集索引**

- 若一个主键被定义，该主键则作为密集索引
- 若没有主键被定义，该表的第一个唯一非空索引则作为密集索引
- 若不满足以上条件，innodb 内部会生成一个隐藏主键（密集索引）
- 非主键索引存储相关键位和其对应的主键值，包含两次查找

<br>

## 二、如何定位并优化慢查询 Sql

具体场景要具体分析，大致思路如下：
- 根据慢日志定位慢查询 sql
- 使用 explain 等工具分析 sql
- 修改 sql 或者尽量让 sql 走索引

### 2.1 根据慢日志定位慢查询 sql

```sql
# 查看数据库慢日志配置
mysql> show variables like "%quer%";

# 查看慢日志数量
mysql> show status like "%slow_queries%";

# 打开慢日志开关
mysql> set global slow_query_log = on;

# 设置慢日志记录阈值为1s，10s可以说是很慢很慢了，不具有参考价值
# 修改后可能需要重新连接数据库才能看到效果
# 另外，这样的设置在重启MySQL服务之后将会失效！如果要永久生效,需修改my.ini配置文件（Mac OS是my.cnf）
mysql> set global long_query_time = 1;
```

![查看数据库慢日志配置](https://i.loli.net/2019/12/24/NFSpx65OUvuqQhD.png)

### 2.2 使用 explain 等工具分析sql

 `explain` 一般写在select查询语句前，用于描述 MySQL 如何执行查询操作以及 MySQL 成功返回结果集需要执行的行数，如：

```sql
mysql> explain select username from user order by id desc;
```

![explain ](https://i.loli.net/2020/02/16/axvG8RU29bmJVXN.png)

#### `type` 字段

表示 MySQL 找到需要的数据行的方式，由快到慢主要有以下几种：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > **index > all**

其中 **Index** 和 **All** 代表全表扫描，速度相对来说是最慢的，可能需要优化。

#### `Extra` 字段

extra 中出现以下 2 项意味着 MySQL 根本不能使用索引，效率会受到重大影响。应尽可能对此进行优化。

| extra 项        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Using filesort  | 表示 MySQL 会对结果使用一个外部索引排序，而不是从表里按索引次序读到相关内容。可能在内存或者磁盘上进行排序。MySQL 中无法利用索引完成的排序操作称为 “ 文件排序 ” |
| Using temporary | 表示 MySQL 在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by。 |

### 2.3 修改 sql 或者尽量让 sql 走索引

在定位到慢查询，并进行简单分析之后，我们就需要对查询语句进行优化。考研让查询条件直接走有索引的字段，当然也可以给字段加索引，比如：

```sql
mysql> alter table user add index index_name(username);
```

加了索引之后，查询语句的执行应该就会快一些。

MySQL 的查询优化器，虽然很强，但是有时候根据它的准则来使用的索引也并不都是最优的，需要根据具体的项目场景进行选择。比如，使用 force index (key) 就可以强制使用某索引。

</br>

## 三、联合索引的最左匹配原则的成因

对于联合索引，形如 KEY 'index_a_b' ('a','b'), 当我们执行查询 where a='' and b='' 和 where a='' 的时候的时候就都会走 index_a_b 这个联合索引，但是 where b='' 就不会走该联合索引了，即最左匹配原则。

#### 详细规则如下：

1.最左前缀匹配原则，非常重要的原则，mysql 会一直向右匹配直到遇到范围查询（>、<、between、like）就停止匹配，比如 a=3 and b=4 and c>5 and d=6 如果建立（a，b，c，d）顺序的索引，d是用不到索引的，如果建立（a，b，d，c）的索引则都可以用到，a，b，d的顺序可以任意调整。

2.= 和 in 可以乱序，比如 a=1 and b=2 and c=3 建立（a，b，c）索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式

#### 成因

MySQL 创建复合索引的规则是首先对最左边的字段进行排序，在此基础上再对第二个索引字段进行排序，一直继续，构建出形如 B+ 的索引结构。所以，第一个字段是肯定有序的，但是直接使用第二个字段的话是不一定有序的，要第一和第二组合使用才是有序的，第一第二第三组合也是有序的...

![最左匹配原则示意图](https://i.loli.net/2020/02/16/Lqu8eBtoOmkgM7r.png)

</br>

## 四、索引是建立得越多越好吗？

答案当然是否定的，所谓物极必反...

1. 数据量小的表不需要建立索引，建立会增加额外的索引开销
2. 数据变更需要维护索引，因此更多的索引意味着
3. 更多的维护成本更多的索引意味着也需要更多的空间



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
