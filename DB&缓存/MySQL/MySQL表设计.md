- [数字类型](#数字类型)
  - [整形类型](#整形类型)
  - [浮点类型和高精度型](#浮点类型和高精度型)
  - [自增设计](#自增设计)
- [字符串类型](#字符串类型)
  - [CHAR 和 VARCHAR 的定义](#char-和-varchar-的定义)
    - [字符集](#字符集)
    - [排序规则（Collation）](#排序规则collation)
    - [正确修改字符集](#正确修改字符集)
  - [账户密码存储设计](#账户密码存储设计)
- [日期类型](#日期类型)
  - [日期类型](#日期类型-1)
    - [DATETIME](#datetime)
    - [TIMESTAMP](#timestamp)
- [非结构存储： 用好JSON](#非结构存储-用好json)
  - [JSON数据类型](#json数据类型)
- [表压缩](#表压缩)
  - [MySQL 压缩表设计](#mysql-压缩表设计)
    - [COMPRESS 页压缩](#compress-页压缩)
    - [TPC压缩](#tpc压缩)
  - [表压缩在业务上的使用](#表压缩在业务上的使用)

# 数字类型

- 不推荐使用整型类型的属性 Unsigned，若非要使用，参数 sql_mode 务必额外添加上选项 NO_UNSIGNED_SUBTRACTION；
- 自增整型类型做主键，务必使用类型 BIGINT，而非 INT，后期表结构调整代价巨大；
- MySQL 8.0 版本前，自增整型会有回溯问题，**做业务开发的你一定要了解这个问题；**
- 当达到自增整型类型的上限值时，再次自增插入，MySQL 数据库会报重复错误；
- 不要再使用浮点类型 Float、Double，MySQL 后续版本将不再支持上述两种类型；
- 账户余额字段，设计是用整型类型（分单位存储），而不是 DECIMAL 类型，这样性能更好，存储更紧凑。

## 整形类型

MySQL 数据库支持 SQL 标准支持的整型类型：INT、SMALLINT。此外，MySQL 数据库也支持诸如 TINYINT、MEDIUMINT 和 BIGINT 整型类型（表 1 显示了各种整型所占用的存储空间及取值范围）：

![](https://secure2.wostatic.cn/static/qfXYQhgeGsMQyNyCKw23EF/image.png?auth_key=1705912606-28BBRgoPzi9er7V2JJcyJE-0-f18121937d20bc4e87738b94d111a84b)

## 浮点类型和高精度型

float

double

DECIMAL

DECIMAL(8,2)  8 是精度（精度表示保存值的主要位数），2 是标度（标度表示小数点后面保存的位数）。

## 自增设计

- 用 BIGINT 做主键，而不是 INT；
- 自增值并不持久化，可能会有回溯现象（MySQL 8.0 版本前）。

在删除自增为 3 的这条记录后，下一个自增值依然为 4（AUTO_INCREMENT=4），这里并没有错误，自增并不会进行回溯。但若这时数据库发生重启，那数据库启动后，表 t 的自增起始值将再次变为 3，即自增值发生回溯。

若要彻底解决这个问题，有以下 2 种方法：

- 升级 MySQL 版本到 8.0 版本，每张表的自增值会持久化；
- 若无法升级数据库版本，则强烈不推荐在核心业务表中使用自增数据类型做主键。

# 字符串类型

https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/MySQL%E5%AE%9E%E6%88%98%E5%AE%9D%E5%85%B8/02%20%20%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B1%BB%E5%9E%8B%EF%BC%9A%E4%B8%8D%E8%83%BD%E5%BF%BD%E7%95%A5%E7%9A%84%20COLLATION.md

MySQL 数据库的字符串类型有 CHAR、VARCHAR、BINARY、BLOB、TEXT、ENUM、SET。不同的类型在业务设计、数据库性能方面的表现完全不同，其中最常使用的是 CHAR、VARCHAR。

- CHAR 和 VARCHAR 虽然分别用于存储定长和变长字符，但对于变长字符集（如 GBK、UTF8MB4），其本质是一样的，都是变长，**设计时完全可以用 VARCHAR 替代 CHAR；**
- 推荐 MySQL 字符集默认设置为 UTF8MB4，可以用于存储 emoji 等扩展字符；
- 排序规则很重要，用于字符的比较和排序，但大部分场景不需要用区分大小写的排序规则；
- 修改表中已有列的字符集，使用命令 ALTER TABLE … CONVERT TO ….；
- 用户性别，运行状态等有限值的列，MySQL 8.0.16 版本直接使用 CHECK 约束机制，之前的版本可使用 ENUM 枚举字符串类型，外加 SQL_MODE 的严格模式；
- 业务隐私信息，如密码、手机、信用卡等信息，需要加密。切记简单的MD5算法是可以进行暴力破解，并不安全，推荐使用动态盐+动态加密算法进行隐私数据的存储。

## CHAR 和 VARCHAR 的定义

CHAR(N) 用来保存固定长度的字符，N 的范围是 0 ~ 255，**请牢记，N 表示的是字符，而不是字节**。VARCHAR(N) 用来保存变长字符，N 的范围为 0 ~ 65536， N 表示字符。在超出 65536 个字符的情况下，可以考虑使用更大的字符类型 TEXT 或 BLOB，两者最大存储长度为 4G，其区别是 BLOB 没有字符集属性，纯属二进制存储。

和 Oracle、Microsoft SQL Server 等传统关系型数据库不同的是，MySQL 数据库的 VARCHAR 字符类型，最大能够存储 65536 个字符，**所以在 MySQL 数据库下，绝大部分场景使用类型 VARCHAR 就足够了。**

### 字符集

**推荐把 MySQL 的默认字符集设置为 UTF8MB4**，否则，某些 emoji 表情字符无法在 UTF8 字符集下存储，比如 emoji 笑脸表情，对应的字符编码为 0xF09F988E

CHAR(1) 既可以存储 1 个 ‘a’ 字节，也可以存储 4 个字节的 emoji 笑脸表情，因此 CHAR 本质也是变长的。

**鉴于目前默认字符集推荐设置为 UTF8MB4，所以在表结构设计时，可以把 CHAR 全部用 VARCHAR 替换，底层存储的本质实现一模一样**。

![](https://secure2.wostatic.cn/static/aCqtkuvwyiHNJBgkAH1KUr/image.png?auth_key=1705912606-83XTQpaow8yp1FqaHnmNNa-0-e6b6e086c44f5797ac986ae1849fbbc4)

### 排序规则（Collation）

**排序规则（Collation）是比较和排序字符串的一种规则**，每个字符集都会有默认的排序规则，你可以用命令 SHOW CHARSET 来查看：

![](https://secure2.wostatic.cn/static/uFt4xNDX4bUdP7zWGgbzoy/image.png?auth_key=1705912606-hHFN3VYw1QNCFMkuNLuAeZ-0-cc0d43dbf063763818f5a1988dcae701)

排序规则以 _ci 结尾，表示不区分大小写（Case Insentive），_cs 表示大小写敏感，_bin 表示通过存储字符的二进制进行比较。需要注意的是，比较 MySQL 字符串，默认采用不区分大小的排序规则

### 正确修改字符集

```Python
mysql> ALTER TABLE emoji_test CONVERT TO CHARSET utf8mb4;

```

## 账户密码存储设计

相信不少开发开发同学会通过函数 MD5 加密存储隐私数据，这没有错，因为 MD5 算法并不可逆。然而，MD5 加密后的值是固定的，如密码 12345678，它对应的 MD5 固定值即为 25d55ad283aa400af464c76d713c07ad。

所以，在设计密码存储使用，还需要加盐（salt），每个公司的盐值都是不同的，因此计算出的值也是不同的。若盐值为 psalt，则密码 12345678 在数据库中的值为：

```Python
password = MD5（‘psalt12345678’）

```

这样的密码存储设计是一种固定盐值的加密算法，其中存在三个主要问题：

- 若 salt 值被（离职）员工泄漏，则外部黑客依然存在暴利破解的可能性；
- 对于相同密码，其密码存储值相同，一旦一个用户密码泄漏，其他相同密码的用户的密码也将被泄漏；
- 固定使用 MD5 加密算法，一旦 MD5 算法被破解，则影响很大。

所以一个真正好的密码存储设计，应该是：动态盐 + 非固定加密算法。

```Python
$salt$cryption_algorithm$value

```

- $salt：表示动态盐，每次用户注册时业务产生不同的盐值，并存储在数据库中。若做得再精细一点，可以动态盐值 + 用户注册日期合并为一个更为动态的盐值。
- $cryption_algorithm：表示加密的算法，如 v1 表示 MD5 加密算法，v2 表示 AES256 加密算法，v3 表示 AES512 加密算法等。
- $value：表示加密后的字符串。

```Shell
SELECT * FROM User\G

*************************** 1. row ***************************

      id: 1

    name: David

     sex: M

password: $fgfaef$v1$2198687f6db06c9d1b31a030ba1ef074

 regDate: 2020-09-07 15:30:00

*************************** 2. row ***************************

      id: 2

    name: Amy

     sex: F

password: $zpelf$v2$0x860E4E3B2AA4005D8EE9B7653409C4B133AF77AEF53B815D31426EC6EF78D882

 regDate: 2020-09-07 17:28:00
```

# 日期类型

- MySQL 5.6 版本开始 DATETIME 和 TIMESTAMP 精度支持到毫秒；
- DATETIME 占用 8 个字节，TIMESTAMP 占用 4 个字节，DATETIME(6) 依然占用 8 个字节，TIMESTAMP(6) 占用 7 个字节；
- TIMESTAMP 日期存储的上限为 2038-01-19 03:14:07，业务用 TIMESTAMP 存在风险；
- 使用 TIMESTAMP 必须显式地设置时区，不要使用默认系统时区，否则存在性能问题，推荐在配置文件中设置参数 time_zone = ‘+08:00’；
- 推荐日期类型使用 DATETIME，而不是 TIMESTAMP 和 INT 类型；
- 表结构设计时，每个核心业务表，推荐设计一个 last_modify_date 的字段，用以记录每条记录的最后修改时间。

## 日期类型

### DATETIME

类型 DATETIME 最终展现的形式为：YYYY-MM-DD HH：MM：SS，固定占用 8 个字节。

从 MySQL 5.6 版本开始，DATETIME 类型支持毫秒，DATETIME(N) 中的 N 表示毫秒的精度。例如，DATETIME(6) 表示可以存储 6 位的毫秒值。同时，一些日期函数也支持精确到毫秒

看情况是否需要存时区，前端给时区，后端数据库存时间一律UTC+8

```Shell
CREATE TABLE User (

    id BIGINT NOT NULL AUTO_INCREMENT,

    name VARCHAR(255) NOT NULL,

    sex CHAR(1) NOT NULL,

    password VARCHAR(1024) NOT NULL,

    money INT NOT NULL DEFAULT 0,

    register_date DATETIME(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),

    last_modify_date DATETIME(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),

    CHECK (sex = 'M' OR sex = 'F'),

    PRIMARY KEY(id)

);

```

### TIMESTAMP

除了 DATETIME，日期类型中还有一种 TIMESTAMP 的时间戳类型，其实际存储的内容为‘1970-01-01 00:00:00’到现在的毫秒数。在 MySQL 中，由于类型 TIMESTAMP 占用 4 个字节，因此其存储的时间上限只能到‘2038-01-19 03:14:07’。

同类型 DATETIME 一样，从 MySQL 5.6 版本开始，类型 TIMESTAMP 也能支持毫秒。与 DATETIME 不同的是，**若带有毫秒时，类型 TIMESTAMP 占用 7 个字节，而 DATETIME 无论是否存储毫秒信息，都占用 8 个字节**。

# 非结构存储： 用好JSON

- 使用 JSON 数据类型，推荐用 MySQL 8.0.17 以上的版本，性能更好，同时也支持 Multi-Valued Indexes；
- JSON 数据类型的好处是无须预先定义列，数据本身就具有很好的描述性；
- 不要将有明显关系型的数据用 JSON 存储，如用户余额、用户姓名、用户身份证等，这些都是每个用户必须包含的数据；
- JSON 数据类型推荐使用在不经常更新的静态数据存储。

## JSON数据类型

JSON（JavaScript Object Notation）主要用于互联网应用服务之间的数据交换。MySQL 支持[RFC 7159](https://tools.ietf.org/html/rfc7159?fileGuid=xxQTRXtVcqtHK6j8)定义的 JSON 规范，主要有**JSON 对象**和**JSON 数组**两种类型。

MySQL 配套提供了丰富的 JSON 字段处理函数，用于方便地操作 JSON 数据，具体可以见 MySQL 官方文档。

其中，最常见的就是函数 JSON_EXTRACT，它用来从 JSON 数据中提取所需要的字段内容，如下面的这条 SQL 语句就查询用户的手机和微信信息。

```SQL
SELECT

    userId,

    JSON_UNQUOTE(JSON_EXTRACT(loginInfo,"$.cellphone")) cellphone,

    JSON_UNQUOTE(JSON_EXTRACT(loginInfo,"$.wxchat")) wxchat

FROM UserLogin;

+--------+-------------+--------------+

| userId | cellphone   | wxchat       |

+--------+-------------+--------------+

|      1 | 13918888888 | 破产码农     |

|      2 | 15026888888 | NULL         |

+--------+-------------+--------------+

2 rows in set (0.01 sec)

 和上面一样的效果
SELECT

    userId,

    loginInfo->>"$.cellphone" cellphone,

    loginInfo->>"$.wxchat" wxchat

FROM UserLogin;

```

当 JSON 数据量非常大，用户希望对 JSON 数据进行有效检索时，可以利用 MySQL 的**函数索引**功能对 JSON 中的某个字段进行索引。

比如在上面的用户登录示例中，假设用户必须绑定唯一手机号，且希望未来能用手机号码进行用户检索时，可以创建下面的索引：

```Python
ALTER TABLE UserLogin ADD COLUMN cellphone VARCHAR(255) AS (loginInfo->>"$.cellphone");

ALTER TABLE UserLogin ADD UNIQUE INDEX idx_cellphone(cellphone);
```

上述 SQL 首先创建了一个虚拟列 cellphone，这个列是由函数 loginInfo->>“$.cellphone” 计算得到的。然后在这个虚拟列上创建一个唯一索引 idx_cellphone。这时再通过虚拟列 cellphone 进行查询，优化器会使用到新创建的 idx_cellphone 索引

# 表压缩

很多同学不会在表结构设计之初就考虑存储的设计，只有当业务发展到一定规模才会意识到问题的严重性。而物理存储主要是考虑是否要启用表的压缩功能，默认情况下，所有表都是非压缩的。

但一些同学一听到压缩，总会下意识地认为压缩会导致 MySQL 数据库的性能下降。**这个观点说对也不对，需要根据不同场景进行区分**

- MySQL 中的压缩都是基于页的压缩；
- COMPRESS 页压缩适合用于性能要求不高的业务表，如日志、监控、告警表等；
- COMPRESS 页压缩内存缓冲池存在压缩和解压的两个页，会严重影响性能；
- 对存储有压缩需求，又希望性能不要有明显退化，推荐使用 TPC 压缩；
- 通过 ALTER TABLE 启用 TPC 压缩后，还需要执行命令 OPTIMIZE TABLE 才能立即完成空间的压缩。

借助页压缩技术，MySQL 可以把一个 16K 的页压缩为 8K，甚至 4K，这样在从磁盘写入或读取时，就能将 I/O 请求大小减半，甚至更小，从而提升数据库的整体性能。

## MySQL 压缩表设计

### COMPRESS 页压缩

COMPRESS 页压缩是 MySQL 5.7 版本之前提供的页压缩功能。只要在创建表时指定ROW_FORMAT=COMPRESS，并设置通过选项 KEY_BLOCK_SIZE 设置压缩的比例。

下面这是一张日志表，ROW_FROMAT 设置为 COMPRESS，表示启用 COMPRESS 页压缩功能，KEY_BLOCK_SIZE 设置为 8，表示将一个 16K 的页压缩为 8K。

```Python
CREATE TABLE Log (

  logId BINARY(16) PRIMARY KEY,

  ......

)

ROW_FORMAT=COMPRESSED

KEY_BLOCK_SIZE=8

```

![](https://secure2.wostatic.cn/static/7DMh1WQB5zgo9H98TN2Qz9/image.png?auth_key=1705912607-jZso2ed4SsCJTXLyN7nZuc-0-147813b6b2625006e49b1474cbd18a93)

COMPRESS 页压缩，适合用于一些对性能不敏感的业务表，例如日志表、监控表、告警表等，压缩比例通常能达到 50% 左右。

虽然 COMPRESS 压缩可以有效减小存储空间，但 COMPRESS 页压缩的实现对性能的开销是巨大的，性能会有明显退化。主要原因是一个压缩页在内存缓冲池中，存在压缩和解压两个页。Page1 和 Page2 都是压缩页 8K，但是在内存中还有其解压后的 16K 页。这样设计的原因是 8K 的页用于后续页的更新，16K 的页用于读取，这样读取就不用每次做解压操作了。

![](https://secure2.wostatic.cn/static/bTV34v8Dq13jqGdRQRY9s/image.png?auth_key=1705912607-uZxm3QY1nCBCps2UMuigLL-0-19d2db20d20c7948995c6da0ae0933c8)

为了 解决压缩性能下降的问题，从MySQL 5.7 版本开始推出了 TPC 压缩功能。

### TPC压缩

TPC（Transparent Page Compression）是 5.7 版本推出的一种新的页压缩功能，其利用文件系统的空洞（Punch Hole）特性进行压缩。可以使用下面的命令创建 TPC 压缩表：

```Python
CREATE TABLE Transaction （

  transactionId BINARY(16) PRIMARY KEY,

  .....

)

COMPRESSION=ZLIB | LZ4 | NONE;
```

要使用 TPC 压缩，首先要确认当前的操作系统是否支持空洞特性。通常来说，当前常见的 Linux 操作系统都已支持空洞特性。

由于空洞是文件系统的一个特性，利用空洞压缩只能压缩到文件系统的最小单位 4K，且其页压缩是 4K 对齐的。比如一个 16K 的页，压缩后为 7K，则实际占用空间 8K；压缩后为 3K，则实际占用空间是 4K；若压缩后是 13K，则占用空间依然为 16K。

![](https://secure2.wostatic.cn/static/4ABn4XNTLaeD54guAeSyAP/image.png?auth_key=1705912607-kqukn8zsU5HBUTMJp63ueA-0-4049c153661cd84237a04df6e25e8876)

上图可以看到，一个 16K 的页压缩后是 8K，接着数据库会对这 16K 的页剩余的 8K 填充0x00，这样当这个 16K 的页写入到磁盘时，利用文件系统空洞特性，则实际将仅占用 8K 的物理存储空间。

空洞压缩的另一个好处是，它对数据库性能的侵入几乎是无影响的（小于 20%），甚至可能还能有性能的提升。

这是因为不同于 COMPRESS 页压缩，TPC 压缩在内存中只有一个 16K 的解压缩后的页，对于缓冲池没有额外的存储开销。

另一方面，所有页的读写操作都和非压缩页一样，没有开销，只有当这个页需要刷新到磁盘时，才会触发页压缩功能一次。但由于一个 16K 的页被压缩为了 8K 或 4K，其实写入性能会得到一定的提升。

## 表压缩在业务上的使用

总的来说，对一些对性能不敏感的业务表，例如日志表、监控表、告警表等，它们只对存储空间有要求，因此可以使用 COMPRESS 页压缩功能。

在一些较为核心的流水业务表上，我更推荐使用 TPC压缩。因为流水信息是一种非常核心的数据存储业务，通常伴随核心业务。如一笔电商交易，用户扣钱、下单、记流水，这就是一个核心业务的微模型。

所以，用户对流水表有性能需求。此外，流水又非常大，启用压缩功能可更为有效地存储数据。

若对压缩产生的性能抖动有所担心，**我的建议**：由于流水表通常是按月或天进行存储，对当前正在使用的流水表不要启用 TPC 功能，对已经成为历史的流水表启用 TPC 压缩功能，如下所示：

![](https://secure2.wostatic.cn/static/55TParNNpDerfc7aJwzmEA/image.png?auth_key=1705912692-9XWW9ncMb57hvZSATLAjbf-0-c8e5c1342d620bec0ecd3689d4c552dd)

