# MySQL 笔记

MySQL 是目前 Web 开发中使用最广泛的**关系型数据库管理系统（RDBMS）**，最初由瑞典 MySQL AB 公司开发，后被 Sun 公司收购，Sun 又被 Oracle 收购，现属于 Oracle 旗下产品。
本笔记以 **MySQL 8.0** 为主（同时标注 5.7 的差异），涵盖数据库基础概念、安装配置与客户端连接、数据类型与约束、数据库与表的操作（DDL）、数据增删改（DML）、数据库设计（三范式与 E-R 模型）、查询（条件、排序、聚合、分组、窗口函数、分页、连接、自关联、子查询）、视图、事务、索引、账户与权限管理、主从同步等核心内容，适合系统性学习与日常查阅。

> **核心理念**：数据库是一种**特殊的文件**，其中按照“**库 → 表 → 行（记录）/ 列（字段）**”的层次存储数据；**SQL** 是操作所有关系型数据库的通用语言。
> 对 Web 程序员来说，重点是熟练编写数据的 **CRUD（增删改查）**：`DQL`（查询）和 `DML`（增删改）必须熟练，`DDL`（建库建表）要会写，`TPL`（事务）、`DCL`（权限）、`CCL`（游标）了解即可。
> **学习目标**：熟练编写数据的增删改查相关的 SQL 语句；在 Python 中操作数据库，本质也是通过 SQL 语句完成的。

> **本笔记的书写约定**：
>
> - Shell 命令（在 Linux / Windows 终端里敲的命令）每行以 `$` 开头，`$` 本身不要输入
> - SQL 语句（在 `mysql>` 客户端里敲的语句）直接书写，**必须以英文分号 `;` 结尾**；`-- ` 开头是 SQL 单行注释（注意 `--` 后面要有一个空格）
> - 表名、字段名、关键字写在代码块中；SQL **关键字不区分大小写**（`SELECT` 与 `select` 等价），但**表名、数据库名是否区分大小写取决于操作系统**（Linux 区分，Windows 不区分），建议统一使用小写

| 章节  | 内容                                                    |
| --- | ----------------------------------------------------- |
| 一   | 数据库基础概念：数据存储的演进、数据库的核心元素、RDBMS、SQL 分类、Python 中的 SQL |
| 二   | MySQL 简介与安装配置：特点、Ubuntu 安装、服务管理、配置文件、密码问题 |
| 三   | 客户端连接：命令行客户端、允许远程连接 |
| 四   | 数据完整性：数据类型（数值、字符串、日期时间、枚举）与约束（主键、非空、唯一、默认、外键） |
| 五   | 数据库与数据表操作（DDL）：库的创建/删除、表的创建/修改/删除、查看表结构 |
| 六   | 数据增删改（DML）：insert、update、delete、逻辑删除、备份与恢复 |
| 七   | 数据库设计：三范式、E-R 模型、三种对应关系与外键 |
| 八   | 查询准备：测试库、测试表与测试数据 |
| 九   | 基础查询：查询列、别名、去重 |
| 十   | 条件查询：比较运算符、逻辑运算符、模糊查询、范围查询、空判断、优先级 |
| 十一  | 排序：order by、asc/desc、多列排序 |
| 十二  | 聚合函数：count、max、min、sum、avg |
| 十三  | 分组：group by、group_concat、having、with rollup |
| 十四  | 窗口函数（8.0 新增）：rank、dense_rank、row_number |
| 十五  | 分页：limit、总页数与第 n 页的计算 |
| 十六  | 连接查询：内连接、左连接、右连接 |
| 十七  | 自关联：省市区数据的单表设计 |
| 十八  | 子查询：标量、列、行子查询 |
| 十九  | SELECT 完整语法与执行顺序 |
| 二十  | 视图：定义、使用、删除与作用 |
| 二十一 | 事务：ACID 特性、事务命令、回滚与提交演示、隔离级别 |
| 二十二 | 索引：原理、使用、性能对比、注意事项 |
| 二十三 | 账户管理与权限：创建账户、授权、改密、远程登录、删除账户 |
| 二十四 | 主从同步：原理、好处、详细配置步骤 |
| 二十五 | 速查：常用命令与语法总览 |

---

## 一、数据库基础概念

### 1.1 数据存储的演进

> 人类记录数据的方式经历了：结绳记事 → 甲骨刻字 → 图书纸质记录 → 文件 → 数据库。

**传统纸质记录数据的缺点**：

- 不易保存
- 备份困难
- 查找不便

**现代化手段一：文件**

- 使用简单，例如 Python 中用 `open` 打开文件，用 `read` / `write` 读写，用 `close` 关闭
- 对于**数据容量较大**的数据，不能很好满足，而且性能较差
- 不易扩展

**现代化手段二：数据库**

- 持久化存储（数据保存在磁盘上，断电不丢失）
- 读写速度极高
- 保证数据的有效性（通过数据类型与约束）
- 对程序支持性非常好，容易扩展

> **类比**：数据库就像一座仓库。真实的仓库里货物按区域、货架分类存放（对应库、表）；而顾客看到的只是整理好的商品列表（对应查询出来的结果集）。

### 1.2 数据库的核心元素

> **数据库**就是一种特殊的文件，其中存储着需要的数据。**关系型数据库**有四个核心元素：
>
> ---

| 元素 | 别名 | 含义 | 类比 Excel |
| --- | --- | --- | --- |
| 数据行 | 记录 | 一条完整的数据 | 一行 |
| 数据列 | 字段 | 数据的某一个属性 | 一列 |
| 数据表 | — | 数据行的集合 | 一个工作表 |
| 数据库 | — | 数据表的集合 | 一个工作簿 |

> ---
>
> 一个数据库就是一个**完整的业务单元**，可以包含多张表，数据被存储在表中。

### 1.3 RDBMS：关系型数据库管理系统

> **RDBMS**（Relational Database Management System）：通过**表**来表示数据之间的关系。所谓关系型数据库，是建立在**关系模型**基础上的数据库，借助集合代数等数学概念和方法来处理数据。
>
> 当前主要使用两种类型的数据库：**关系型数据库**与**非关系型数据库**（NoSQL，后续学习）。本笔记只讨论关系型数据库。
>
> 数据库排名可查看：[db-engines.com/en/ranking](https://db-engines.com/en/ranking)

| 产品 | 特点与应用场景 |
| --- | --- |
| Oracle | 以前的大型项目中使用，如银行、电信等 |
| MySQL | Web 时代使用最广泛的关系型数据库 |
| MS SQL Server | 微软的项目中使用 |
| SQLite | 轻量级数据库，主要应用在移动平台 |
| DB2 | IBM 的关系型数据库 |

> **RDBMS 与数据库的关系**：RDBMS（如 MySQL 软件）是管理数据库的**软件**，一个 RDBMS 中可以创建**多个数据库**，每个数据库中可以有**多张表**。
> 即：`MySQL 服务器（RDBMS）` ⊃ `数据库` ⊃ `数据表` ⊃ `数据行 / 数据列`。

### 1.4 SQL：结构化查询语言

> **SQL**（Structured Query Language，结构化查询语言）是一种用来操作 RDBMS 的数据库语言。当前所有关系型数据库都支持使用 SQL 进行操作，也就是说可以通过 SQL 操作 Oracle、SQL Server、MySQL、SQLite 等所有关系型数据库（各产品之间存在少量“方言”差异）。
>
> SQL **不区分大小写**（指关键字）。
>
> **SQL 语句主要分为六类：**
>
> ---

| 分类 | 全称 | 作用 | 典型语句 |
| --- | --- | --- | --- |
| DQL | 数据查询语言 | 对数据进行查询 | `select` |
| DML | 数据操作语言 | 对数据进行增加、修改、删除 | `insert`、`update`、`delete` |
| TPL | 事务处理语言 | 对事务进行处理 | `begin transaction`、`commit`、`rollback` |
| DCL | 数据控制语言 | 进行授权与权限回收 | `grant`、`revoke` |
| DDL | 数据定义语言 | 数据库、表的管理等 | `create`、`drop`、`alter` |
| CCL | 指针（游标）控制语言 | 通过控制指针完成表的操作 | `declare cursor` |

> ---
>
> 对于 Web 程序员来讲，重点是数据的 **CRUD（增删改查）**：必须熟练编写 `DQL`、`DML`，能够编写 `DDL` 完成数据库、表的操作，其他的 `TPL`、`DCL`、`CCL` 了解即可。
>
> **CRUD** 的含义：**C**reate（创建）、**R**etrieve（读取）、**U**pdate（更新）、**D**elete（删除）。

### 1.5 在 Python 中执行 SQL

> 在 Python 代码中操作数据，本质就是**把 SQL 语句以字符串形式交给数据库执行**。下面用第三方库 `pymysql` 演示（先 `pip install pymysql`）。

```python
from pymysql import connect  # 导入 connect 函数，用于创建数据库连接


def main():
    """连接 MySQL，查询 id 为 7 的学生，并打印查询结果。"""
    # 创建 Connection 连接对象
    # host：服务器地址（str）；port：端口（int，MySQL 默认 3306）
    # user / password：登录账号与密码（str）；database：要使用的库名（str）
    # charset：连接使用的字符集（str），要与库的字符集一致，否则会乱码
    conn = connect(host='localhost', port=3306, user='root',
                   password='mysql', database='python1', charset='utf8')
    # 获取 Cursor（游标）对象，所有 SQL 都通过它来执行
    cs = conn.cursor()
    # SQL 语句：%s 是占位符，由 execute 的第二个参数填充
    # 不要用字符串拼接的方式拼 SQL，否则会产生 SQL 注入风险
    sql = 'select id,name from students where id = %s'
    # execute(sql, 参数元组)：返回 int，表示受影响（或查询到）的行数
    count = cs.execute(sql, (7,))
    print(count)  # 打印受影响的行数，int
    # fetchall()：取出全部结果，返回 tuple，每个元素也是 tuple（一行数据）
    print(cs.fetchall())
    # 如果执行的是 insert / update / delete，必须调用 conn.commit() 才会生效
    cs.close()  # 关闭游标
    conn.close()  # 关闭连接


if __name__ == '__main__':
    main()
```

---

## 二、MySQL 简介与安装配置

### 2.1 MySQL 简介

> [MySQL 官网](https://www.mysql.com/)。MySQL 是一个关系型数据库管理系统，开源免费、使用范围广、跨平台支持性好、提供多种语言调用的 API，是学习数据库开发的首选。

| 特点 | 说明 |
| --- | --- |
| 可移植性 | 使用 C 和 C++ 编写，并使用多种编译器测试，保证源码可移植 |
| 跨平台 | 支持 Linux、Windows、AIX、FreeBSD、HP-UX、macOS、Solaris 等多种操作系统 |
| 多语言 API | 为 C、C++、Python、Java、Perl、PHP、Eiffel、Ruby 等提供 API |
| 多线程 | 支持多线程，充分利用 CPU 资源 |
| 查询优化 | 优化的 SQL 查询算法，有效提高查询速度 |
| 多字符集 | 支持 GB2312、BIG5、UTF-8 等多种编码 |
| 多种连接途径 | 提供 TCP/IP、ODBC 和 JDBC 等 |
| 管理工具 | 提供用于管理、检查、优化数据库操作的管理工具 |
| 大数据量 | 可以处理拥有上千万条记录的大型数据库 |
| 多存储引擎 | 支持多种存储引擎（如 InnoDB、MyISAM） |
| 双授权 | 分为**社区版**和**商业版**，体积小、速度快、总体拥有成本低，中小型网站首选 |
| 标准 SQL | 使用标准的 SQL 数据语言形式 |
| 可定制 | 采用 GPL 协议，可以修改源码开发自己的 MySQL 系统 |

> 此外，MySQL 还提供在线 DDL 更改、复制全局事务标识、复制无崩溃从机、复制多线程从机等高级功能。

### 2.2 服务器端安装（Ubuntu）

> MySQL 采用 **C/S 架构（客户端/服务器）**：**服务器端**负责接收客户端的请求、执行 SQL 语句、管理数据库；**客户端**（命令行 `mysql`、图形化工具等）负责连接服务器并发送 SQL。两者通过 socket（TCP/IP）通信。

**第 1 步（仅 Ubuntu 18.04 需要，20.04 不需要）：配置 MySQL 官方 apt 源**

```bash
# 把官方提供的 mysql-apt-config_0.8.12-1_all.deb 放到 Ubuntu 下面，然后安装它
# dpkg -i：安装本地 .deb 软件包
# 安装过程中弹出选择界面：第一项（MySQL Server）进去改成 8.0，再选 OK
$sudo dpkg -i mysql-apt-config_0.8.12-1_all.deb
```

**第 2 步：安装服务器端**

```bash
# 一定要先更新软件源索引，再安装；如果 update 报错，需要先解决 Ubuntu 的源问题
$sudo apt update

# 安装 MySQL 服务器端，安装过程中按提示设置 root 密码（务必设置，且两次输入一致）
$sudo apt install mysql-server
```

> **安装提示**：安装 8.0 时会弹出认证方式选择：
>
> 1. `Use Strong Password Encryption (RECOMMENDED)`：使用强密码加密（`caching_sha2_password`，8.0 默认）
> 2. `Use Legacy Authentication Method (Retain MySQL 5.x Compatibility)`：使用旧版认证方式（`mysql_native_password`，兼容 5.x）
>
> 为了兼容旧版客户端（如旧版图形化工具、旧版驱动），**选择 2**。
>
> 如果安装时没有设置密码，可参考 [2.5 密码问题](#25-密码问题与重置) 进行修改。

**第 3 步：安装客户端（按需）**

```bash
# 刷新软件包索引：索引过期会导致安装时报 404 Not Found，安装前先执行
$sudo apt update

# 安装命令行客户端：如果 mysql -u root -p 提示找不到命令，就需要安装它
$sudo apt install mysql-client

# 验证安装：-V（等同于 --version）输出客户端版本号，能输出说明客户端安装成功
$mysql -V
```

### 2.3 服务管理

> 服务器端一般以**服务**方式管理，服务名称为 `mysql`。

```bash
# 启动 MySQL 服务（如果登录时报“无法连接到 socket”之类的错误，说明服务没启动）
$sudo service mysql start

# 查看服务状态（active (running) 表示正在运行）
$sudo service mysql status

# 查看进程中是否存在 mysql 服务
# ps -elf：列出所有进程的详细信息；grep mysql：过滤出包含 mysql 的行
$ps -elf | grep mysql

# 停止服务
$sudo service mysql stop

# 重启服务（修改配置文件后必须重启才能生效）
$sudo service mysql restart
```

### 2.4 配置文件

> Ubuntu 下配置文件入口为 `/etc/mysql/mysql.cnf`，它会引入两个目录：
>
> - `conf.d` 目录：其中的 `mysql.cnf` 默认没有配置项
> - `mysql.conf.d` 目录：其中的 `mysqld.cnf` 才是**服务器端主要配置**所在
>
> ---

| 配置项 | 含义 | 默认值 |
| --- | --- | --- |
| `bind-address` | 服务器绑定的 IP，决定哪些地址可以连接 | `127.0.0.1`（只允许本机连接） |
| `port` | 端口号 | `3306` |
| `datadir` | 数据库文件存放目录 | `/var/lib/mysql` |
| `general_log_file` | 普通日志文件 | `/var/log/mysql/mysql.log` |
| `log_error` | 错误日志文件 | `/var/log/mysql/error.log` |

> ---
>
> **注意**：修改配置文件后必须 `sudo service mysql restart` 重启服务才会生效。

### 2.5 密码问题与重置

> 以下内容针对**出现异常时**才需要查看（安装时没设置密码、忘记密码等）。

**MySQL 5.7 安装时没配置密码的修改步骤：**

```bash
# 5.7 中用 root（系统管理员）身份可以不输密码直接进入数据库
$sudo mysql
```

```sql
-- 切换到系统库 mysql（用户信息都存放在其中的 user 表）
use mysql;
-- 修改 root 在 localhost 登录的密码；password() 是 5.7 的加密函数（8.0 已删除）
update mysql.user set authentication_string=password('123') where user='root' and host='localhost';
-- 把认证插件改为 mysql_native_password，方便各种客户端连接
update user set plugin="mysql_native_password";
-- 刷新权限表，使修改立即生效（直接修改系统表后必须执行）
flush privileges;
-- 退出客户端
quit;
```

```bash
# 重启 MySQL 服务
$sudo service mysql restart
# 使用新密码登录
$mysql -u root -p
```

**MySQL 8.0（如 Ubuntu 20.04）没有配置密码的修改方式：**

```bash
# 以系统管理员身份直接进入数据库
$sudo mysql
```

```sql
-- 8.0 使用 alter user 修改密码；with mysql_native_password 指定认证插件
-- 'root'@'localhost' 表示“从本机登录的 root 用户”
-- by 后面的 '123' 是要设置的新密码，请替换成自己的密码
alter user 'root'@'localhost' identified with mysql_native_password by '123';
-- 刷新权限
flush privileges;
```

```bash
# 重启 MySQL 服务
$sudo service mysql restart
```

---

## 三、客户端连接

### 3.1 命令行客户端

> 在工作中**主要使用命令行方式**操作数据库，要求熟练编写 SQL。

```bash
# 查看命令行客户端的全部选项和帮助文档
$mysql --help

# 最常用的连接方式：-u 指定用户名，-p 表示接下来提示输入密码（密码不会显示）
$mysql -u root -p

# 带密码直接登录：-p 与密码之间不能有空格（密码会留在命令历史中，不安全，仅学习使用）
$mysql -uroot -p123

# 完整格式：-h 服务器地址，-P（大写）端口
# 最后的 test_db 是数据库名（需已存在），登录后自动进入该库，相当于省去 use test_db;
$mysql -h 127.0.0.1 -P 3306 -u root -p test_db
```

> 连接成功后出现 `mysql>` 提示符，表示已进入客户端，可以输入 SQL。
>
> **退出登录**：输入 `quit` 或 `exit`，或按 `Ctrl + D`。

```sql
-- 查看 MySQL 服务器的版本号
select version();
-- 显示当前的日期和时间
select now();
```

> **客户端使用小技巧**：
>
> - SQL 语句以 `;` 结尾才会执行，可以跨多行书写
> - 语句末尾用 `\G` 代替 `;` 可以把结果**竖着显示**（每个字段一行），适合字段很多的结果，如 `show slave status\G`
> - `source 文件名.sql;` 可以执行 SQL 脚本文件，常用来导入数据

### 3.2 开启远程连接

> 以下方法用于让**其他机器**能连接到本服务器的 MySQL，**仅用于学习环境，生产环境中不要随意开放 root 远程登录**。

**MySQL 8.0：**

```bash
# 以 root 登录
$mysql -u root -p
```

```sql
-- 切换到系统库
use mysql;
-- 查看原有的账户配置：host 表示允许从哪里登录，localhost 表示仅限本机
select host,user from user;
-- 把 root 的 host 改成 %（% 表示任意主机）
update user set host='%' where user='root';
-- 刷新权限
flush privileges;
```

```bash
# 退出 MySQL 后，在系统终端重启服务，使远程访问配置生效
$sudo service mysql restart
```

**MySQL 5.7：**

```sql
-- 授予 root 在任意主机上的全部权限；identified by 后是密码；with grant option 表示可以把权限再授予别人
-- 注意：8.0 中 grant 语句不能再带 identified by，必须先 create user（见 23.3）
grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;
-- 刷新权限
flush privileges;
```

**如果还是连不上**，需要修改服务器的绑定地址：

```bash
# 编辑 MySQL 的服务端配置文件
$sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```
# bind-address 表示服务器绑定的 IP，默认为 127.0.0.1（只允许本机连接）
# 改为 0.0.0.0 表示监听所有网卡，即允许其他机器连接
bind-address = 0.0.0.0
```

```bash
# 修改配置后必须重启服务
$sudo service mysql restart
```

---

## 四、数据完整性：数据类型与约束

> 在表中为了**更加准确地存储数据，保证数据的正确有效**，可以在创建表的时候，为表添加一些强制性的验证，包括：**数据类型**（字段能存什么）和**约束**（字段必须满足什么规则）。

### 4.1 数据类型

> 可以通过查阅帮助文档获取所有支持的数据类型。
>
> **使用数据类型的原则：够用就行**。尽量使用取值范围小的，而不用大的，这样可以节省存储空间、提高效率。
>
> 常用数据类型：
>
> - 整数：`int`、`bit`（1~64 位）
> - 小数：`decimal`
> - 字符串：`varchar`、`char`
> - 日期时间：`date`、`time`、`datetime`
> - 枚举类型：`enum`

#### 4.1.1 数值类型

| 类型 | 字节 | 有符号范围（Signed） | 无符号范围（Unsigned） |
| --- | --- | --- | --- |
| `TINYINT` | 1 | -128 ~ 127 | 0 ~ 255 |
| `SMALLINT` | 2 | -32768 ~ 32767 | 0 ~ 65535 |
| `MEDIUMINT` | 3 | -8388608 ~ 8388607 | 0 ~ 16777215 |
| `INT` / `INTEGER` | 4 | -2147483648 ~ 2147483647 | 0 ~ 4294967295 |
| `BIGINT` | 8 | -9223372036854775808 ~ 9223372036854775807 | 0 ~ 18446744073709551615 |

| 类型 | 说明 |
| --- | --- |
| `BIT(M)` | 位类型。`M` 指定位数，默认值 1，范围 1~64 |
| `BOOL`、`BOOLEAN` | 使用 0 或 1 表示假或真（是 `TINYINT(1)` 的同义词） |
| `FLOAT[(M,D)]` | 单精度浮点数。`M` 指定总位数（显示长度），`D` 指定小数位数 |
| `DOUBLE[(M,D)]` | 双精度浮点数，比 `FLOAT` 精度更大 |
| `DECIMAL(M,D)` | **定点数**（精确小数），`M` 为总位数，`D` 为小数位数，如 `decimal(5,2)` 表示共 5 位数字、其中小数占 2 位，范围 `-999.99 ~ 999.99` |

> **修饰符**：
>
> - `UNSIGNED`：无符号（不允许负数），正数范围翻倍，如 `tinyint unsigned` 范围 `0 ~ 255`
> - `ZEROFILL`：显示时用 0 填充至指定宽度，并自动变为无符号
>
> **注意**：`float` / `double` 是**近似值**，存在精度误差，**涉及金额等需要精确计算的字段必须使用 `decimal`**。PDF 讲义中把 `decimal` 称为“浮点数”，严格来说它是**定点数**，精确存储。

#### 4.1.2 字符串类型（值要用单引号引起来）

| 类型 | 说明 | 示例 |
| --- | --- | --- |
| `CHAR(size)` | **固定长度**字符串，`size` 表示字符个数，范围 0~255 | `char(3)` 存 `'ab'`，实际占 3 个字符的空间（补空格），读取时默认去掉末尾空格 |
| `VARCHAR(size)` | **可变长度**字符串，`size` 表示最大字符个数（理论上限 65535 字节，受行大小限制） | `varchar(3)` 存 `'ab'`，只占 2 个字符的空间 |
| `TEXT` | 大文本，最大 65535 字节；更大的有 `MEDIUMTEXT`、`LONGTEXT`（`LONGTEXT` 最大 4GB） | 文章、评论内容等 |
| `BLOB`、`LONGBLOB` | 二进制数据 | 不推荐存放在数据库中 |

> **特别说明**：
>
> - `char`：长度固定，处理速度快，适合长度固定的数据（如手机号、身份证号、性别、MD5）
> - `varchar`：长度可变，节省空间，适合长度不固定的数据（如姓名、标题）
> - **超长怎么办**：`char(3)` / `varchar(3)` 存入 `'abcd'`，PDF 讲义写“实际存储为 `'abc'`”（截断）。这是**非严格模式**下的行为；MySQL 5.7 及 8.0 默认是**严格模式**，会直接报错 `Data too long for column`，数据不会被写入
> - 字符串 `text` 用于存储大文本，当字符数大于 4000 字节时推荐使用
> - **图片、音频、视频等文件不存储在数据库中**，而是上传到某个服务器（或对象存储）上，然后在表中存储这个文件的**保存路径**

#### 4.1.3 日期时间类型（值要用单引号引起来）

| 类型 | 字节 | 格式示例 | 说明 |
| --- | --- | --- | --- |
| `DATE` | 3 | `'2020-01-01'` | 日期，范围 `1000-01-01 ~ 9999-12-31` |
| `TIME` | 3 | `'12:29:59'` | 时间 |
| `DATETIME` | 8 | `'2020-01-01 12:29:59'` | 日期 + 时间，范围 `1000-01-01 00:00:00 ~ 9999-12-31 23:59:59` |
| `YEAR` | 1 | `'2017'` | 年份 |
| `TIMESTAMP` | 4 | `'1970-01-01 00:00:01'` | 时间戳，范围 `1970-01-01 00:00:01 UTC ~ 2038-01-19 03:14:07 UTC`，可用于**自动记录** insert、update 操作的时间 |

> **说明**：讲义中 `DATE` 写的是 4 字节，实际为 **3 字节**。`DATETIME` 在 5.6.4 以后占 5~8 字节（含小数秒时更多），讲义按 8 字节记。
>
> `TIMESTAMP` 受**时区**影响（存储 UTC、读取时按会话时区转换），且只到 2038 年；`DATETIME` 不受时区影响。
>
> 例如让一个字段自动记录创建时间和最后修改时间：

```sql
-- created：插入时自动取当前时间；updated：每次修改该行时自动更新为当前时间
create table demo_time(
    id int unsigned primary key auto_increment,
    created datetime default current_timestamp,
    updated datetime default current_timestamp on update current_timestamp
);
```

#### 4.1.4 枚举类型 `enum`

> `enum('值1','值2',...)`：字段的值**只能是列出的值之一**（或 `NULL`）。内部按**序号**存储（从 1 开始），所以插入时既可以写字符串，也可以写序号。

```sql
-- gender 只能取 男/女/中性/保密 之一；序号分别是 1、2、3、4
gender enum('男','女','中性','保密')
-- 以下两种写法等价：插入 '女' 或者插入序号 2
-- insert into students(gender) values('女');
-- insert into students(gender) values(2);
```

### 4.2 约束

| 约束 | 关键字 | 说明 |
| --- | --- | --- |
| 主键 | `primary key` | 唯一标识一行数据，**不能重复且不能为空**；InnoDB 中数据按主键顺序**物理存储**（聚集索引）；一张表只能有一个主键 |
| 非空 | `not null` | 此字段不允许填写空值（`NULL`） |
| 唯一 | `unique` | 此字段的值不允许重复（允许有多个 `NULL`） |
| 默认 | `default` | 当不填写此值时会使用默认值；如果填写，以填写的为准 |
| 外键 | `foreign key` | 对**关系字段**进行约束：为关系字段填值时，会到关联的表中查询此值是否存在，存在则填写成功，不存在则填写失败并抛出异常 |
| 自增 | `auto_increment` | 整数字段自动增长，通常和主键搭配；插入时不用填写 |

> **关于外键的说明**：虽然外键约束可以保证数据的有效性，但在进行数据的 CRUD 时，都会**降低数据库的性能**（每次都要检查关联表），所以**实际开发中通常不推荐使用外键**。那么数据的有效性怎么保证呢？**可以在逻辑层（程序代码）中进行控制**。外键的语法见 [7.4](#74-外键的使用)。

---

## 五、数据库与数据表操作（DDL）

### 5.1 数据库操作

```sql
-- 查看当前服务器上的所有数据库
show databases;
-- 使用（切换到）某个数据库，之后的操作都针对它：use 数据库名;
use test_db;
-- 查看当前正在使用的数据库，如果没有选择则返回 NULL
select database();
-- 创建数据库：create database 数据库名 charset=字符集;
-- charset=utf8 指定字符集；utf8 在 MySQL 中最多支持 3 字节字符，如果要存 emoji 表情，应使用 utf8mb4
create database test_db charset=utf8;
-- 查看数据库的创建语句（可看到字符集等信息）
show create database test_db;
-- 删除数据库（会连同库里所有表和数据一起删除，且无法恢复，慎用）
drop database test_db;
```

> **推荐写法**：`create database if not exists test_db charset=utf8mb4;`，加上 `if not exists` 可避免库已存在时报错；删除时同理 `drop database if exists test_db;`。

### 5.2 数据表操作

**查看表**

```sql
-- 查看当前数据库中的所有表
show tables;
-- 查看表结构：显示每个字段的名称、类型、是否为空、键、默认值、额外信息；desc 是 describe 的简写
desc classes;
-- 查看表的创建语句（可看到完整的建表 SQL、引擎、字符集等）
show create table classes;
```

**创建表**

> 语法：
>
> ```
> create table 表名(
>     列名1 数据类型 [约束],
>     列名2 数据类型 [约束],
>     ...
>     [primary key(一个或多个列)]
> );
> ```
>
> `auto_increment` 表示自动增长。

```sql
-- 例：创建班级表 classes
create table classes(
    -- id：无符号整型，自动增长，主键，非空
    id int unsigned auto_increment primary key not null,
    -- name：最长 10 个字符的可变长字符串
    name varchar(10)
);

-- 例：创建学生表 students
create table students(
    -- 主键，自动增长
    id int unsigned primary key auto_increment not null,
    -- 姓名：最长 20 个字符，默认空字符串
    name varchar(20) default '',
    -- 年龄：无符号 tinyint（0~255），默认 0
    age tinyint unsigned default 0,
    -- 身高：共 5 位数字，其中 2 位小数，如 181.50（单位 cm）
    height decimal(5,2),
    -- 性别：枚举，只能是这四个值之一
    gender enum('男','女','中性','保密'),
    -- 所属班级编号：无符号整型，默认 0（对应 classes 表的 id）
    cls_id int unsigned default 0
);
```

**修改表（`alter table`）**

> 语法速记：

| 操作 | 语法 | 说明 |
| --- | --- | --- |
| 添加字段 | `alter table 表名 add 列名 类型;` | 在表的**最后**添加一个字段 |
| 修改字段（可重命名） | `alter table 表名 change 原名 新名 类型及约束;` | 改名并重新定义类型约束，**即使不改名也要写两遍** |
| 修改字段（不重命名） | `alter table 表名 modify 列名 类型及约束;` | 只改类型和约束 |
| 删除字段 | `alter table 表名 drop 列名;` | 删除该字段及其所有数据 |

```sql
-- 添加字段：给 students 增加 birthday（生日，日期时间类型）
alter table students add birthday datetime;
-- 修改字段（重命名版）：把 birthday 改名为 birth，同时设置为 datetime 且不允许为空
-- （建议在空表或没有 NULL 数据时执行，否则非严格模式下旧数据会被转成零值，严格模式下会报错）
alter table students change birthday birth datetime not null;
-- 修改字段（不重命名版）：只修改 birth 的类型为 date，仍然不允许为空
alter table students modify birth date not null;
-- 删除字段：删除 birth 字段（讲义中写的是 drop birthday，但此时字段已改名为 birth，所以这里用 birth）
alter table students drop birth;
```

**删除表**

```sql
-- 删除整张表（结构和数据一起删除，无法恢复）
drop table students;
-- 推荐：表存在才删除，避免报错
drop table if exists students;
-- 查看表的创建语句
show create table classes;
```

> **三个“删除”的区别（补充）**：
>
> - `delete from 表名 [where ...]`：删除**数据行**，可带条件，可回滚（事务中），表结构保留，自增计数不重置
> - `truncate table 表名`：**清空全部数据**，表结构保留，自增计数重置，速度快，不能回滚
> - `drop table 表名`：删除**整张表**（结构 + 数据）

---

## 六、数据增删改（DML）

> 本章对应 CRUD 中的 **C**（增）、**U**（改）、**D**（删），查询 **R** 先看基本用法，详见第八章之后。
>
> 为便于演示，本章假定 `students` 表的结构如下（在 5.2 的基础上增加了 `birth` 列）：

```sql
-- 重新创建演示用的学生表（先删除旧表）
drop table if exists students;
create table students(
    id int unsigned primary key auto_increment not null,
    name varchar(20) default '',
    age tinyint unsigned default 0,
    height decimal(5,2),
    gender enum('男','女','中性','保密'),
    cls_id int unsigned default 0,
    -- 生日：日期类型，允许为空
    birth date
);
-- 如果 classes 表不存在，也需要创建（见 5.2）
```

### 6.1 查询基本使用

```sql
-- 查询所有列：* 表示所有字段（实际开发中不推荐，应写明需要的列）
select * from classes;
-- 查询指定列：多个列用逗号分隔，可以用 as 为列或表指定别名
select id,name from classes;
```

### 6.2 增加数据（insert）

> 格式：`INSERT [INTO] 表名 [(列名,...)] {VALUES | VALUE} ({值 | DEFAULT},...),(...),...`
>
> **说明**：
>
> - **主键列是自动增长的**，但是在**全列插入**时需要“占位”，通常使用 `0`、`default` 或 `null` 来占位，插入成功后以**实际生成的数据**为准
> - 字符串、日期要用**单引号**引起来

```sql
-- 1) 全列插入：值的顺序必须与表中字段的顺序一一对应；id 用 0 占位，由 MySQL 自动生成
insert into students values(0,'郭靖',20,181.00,'男',1,'2016-1-2');
-- 2) 部分列插入：值的顺序与给出的列顺序对应；没写的列使用默认值（没有默认值则为 NULL）
insert into students(name,birth) values('黄蓉','2016-3-2');
-- 3) 多行插入：一条语句插入多行，可以减少与数据库的通信次数，效率更高
-- 3.1 全列多行插入
insert into classes values(0,'python1'),(0,'python2');
-- 3.2 部分列多行插入
insert into students(name) values('杨康'),('杨过'),('小龙女');
```

### 6.3 修改数据（update）

> 格式：`UPDATE 表名 SET 列1={值1|DEFAULT} [,列2={值2|DEFAULT}]... [WHERE 条件]`
>
> **务必写 `where` 条件**，否则会修改表中**所有行**！

```sql
-- 把 id 为 5 的学生的性别改为 2（枚举序号 2，即 '女'）
update students set gender=2 where id=5;
-- 同时修改多列：用逗号分隔
update students set age=21,height=170.50 where id=1;
```

### 6.4 删除数据（delete）与逻辑删除

> 格式：`DELETE FROM 表名 [WHERE 条件]`
>
> **务必写 `where` 条件**，否则会删除表中**所有数据**！

```sql
-- 物理删除：把 id 为 5 的学生从表中彻底删除（无法找回）
delete from students where id=5;
```

> **逻辑删除**：对于重要数据，不希望物理删除（一旦删除就无法找回）。方案是给表加一个标记列（如 `isdelete`，类型 `bit`，默认 0），“删除”时只是把它**改为 1**，查询时只查 `isdelete=0` 的数据。**本质就是一个 update 操作**。
>
> 数据是否重要，要根据实际开发决定；不重要的数据可以直接物理删除。

```sql
-- 给 students 增加逻辑删除标记列：bit 类型，默认 0 表示“未删除”
alter table students add isdelete bit default 0;
-- 逻辑删除：id 为 1 的学生标记为已删除
update students set isdelete=1 where id=1;
-- 查询时只查没被删除的
select * from students where isdelete=0;
```

### 6.5 备份与恢复

> 使用 MySQL 自带的命令行工具 `mysqldump`（注意：是在 **Shell 终端**中运行，不是在 `mysql>` 里）。

**备份**

```bash
# mysqldump：导出数据库为 SQL 脚本文件
# -u 用户名；-p 回车后按提示输入密码；test_db 是要备份的数据库名；> 表示把输出重定向到文件
$mysqldump -uroot -p test_db > test_db.sql
```

**恢复**

```bash
# 第 1 步：连接 MySQL，创建一个新的空数据库（恢复时数据库必须已存在）
$mysql -uroot -p
```

```sql
-- 创建用于恢复的新数据库
create database test_db_new charset=utf8;
-- 退出客户端
quit
```

```bash
# 第 2 步：用 < 把 SQL 文件作为输入交给 mysql 执行，从而重建表和数据
# test_db_new 是刚创建的目标数据库名；根据提示输入密码
$mysql -uroot -p test_db_new < test_db.sql
```

---

## 七、数据库设计

> 关系型数据库建立在 **E-R 模型**的基础上。项目开始的第一步，就是根据产品经理的设计策划，抽取出**模型与关系**，制定出**表结构**。
>
> - 设计数据库的软件很多，常用的有 **Power Designer**、**DB Designer** 等，可以直观地看到实体及实体间的关系
> - 设计工作可能由专门的数据库设计人员完成，也可能由开发组成员完成，一般是项目经理带领组员来完成
> - 现阶段不需要独立完成数据库设计，但要注意积累一些这方面的经验
> - 扩展阅读：看看别人家的规范，如《58 到家数据库 30 条军规解读》

### 7.1 三范式

> 经过研究和对使用中问题的总结，对于设计数据库提出了一些规范，这些规范被称为**范式**（Normal Form）。目前有迹可循的共有 8 种范式，**一般需要遵守前 3 种（三范式）即可**。
>
> ---

| 范式 | 要求 | 一句话理解 |
| --- | --- | --- |
| 第一范式（1NF） | 强调列的**原子性**，即列不能再分成其他几列 | 每个字段都是不可再分的最小单元 |
| 第二范式（2NF） | 首先是 1NF；表必须有主键；**非主键列必须完全依赖于主键**，不能只依赖于主键的一部分 | 消除“部分依赖”（针对联合主键） |
| 第三范式（3NF） | 首先是 2NF；**非主键列必须直接依赖于主键**，不能存在传递依赖 | 消除“传递依赖” |

> ---
>
> **区分 2NF 与 3NF 的关键点**：
>
> - **2NF**：非主键列是**完全依赖于主键**，还是只依赖于主键的**一部分**
> - **3NF**：非主键列是**直接依赖于主键**，还是直接依赖于**另一个非主键列**（传递依赖：非主键列 A 依赖于非主键列 B，B 依赖于主键）

**示例一：1NF**

> 考虑表【联系人】（姓名，性别，电话）。如果一个联系人有家庭电话和公司电话，那么“电话”列就不是原子的，**不符合 1NF**。
>
> 把列（电话）拆分：【联系人】（姓名，性别，家庭电话，公司电话），即符合 1NF。

**示例二：2NF**

> 考虑订单明细表【OrderDetail】（OrderID，ProductID，UnitPrice，Discount，Quantity，ProductName）。
>
> 一个订单中可以订购多种产品，所以单单一个 `OrderID` 不足以成为主键，主键应该是（`OrderID`，`ProductID`）。显然：
>
> - `Discount`（折扣）、`Quantity`（数量）**完全依赖于**主键（`OrderID`，`ProductID`）
> - `UnitPrice`（单价）、`ProductName`（产品名）**只依赖于 `ProductID`**（主键的一部分）
>
> 所以该表**不符合 2NF**，不符合 2NF 的设计容易产生**冗余数据**（同一个产品的单价、名称在每个订单里重复存储）。
>
> **拆分为**：【OrderDetail】（OrderID，ProductID，Discount，Quantity）和【Product】（ProductID，UnitPrice，ProductName），消除 `UnitPrice`、`ProductName` 的多次重复。

**示例三：3NF**

> 考虑订单表【Order】（OrderID，OrderDate，CustomerID，CustomerName，CustomerAddr，CustomerCity），主键是（`OrderID`）。
>
> - `OrderDate`、`CustomerID`、`CustomerName`、`CustomerAddr`、`CustomerCity` 都完全依赖于主键 `OrderID`，所以符合 2NF
> - 但 `CustomerName`、`CustomerAddr`、`CustomerCity` **直接依赖的是 `CustomerID`（非主键列）**，而不是直接依赖于主键，它们是通过传递才依赖于主键，所以**不符合 3NF**
>
> **拆分为**：【Order】（OrderID，OrderDate，CustomerID）和【Customer】（CustomerID，CustomerName，CustomerAddr，CustomerCity），从而达到 3NF。

> **注意**：范式是**指导**而不是铁律。严格遵守范式会导致表变多、查询需要多表连接，有时为了性能会故意**反范式**设计（适当冗余）。

### 7.2 E-R 模型

> - **E**：Entity（实体）。设计实体就像定义一个类一样，指定从哪些方面描述对象，**一个实体转换为数据库中的一个表**
> - **R**：Relationship（关系）。关系描述两个实体之间的对应规则，关系的类型包括**一对一、一对多、多对多**
> - **关系也是一种数据**，需要通过**一个字段**存储在表中

### 7.3 三种关系的实现

| 关系 | 实现方式 | 示例 |
| --- | --- | --- |
| **一对一** | 在表 A 或表 B 中任意一边创建一个字段，存储另一个表的主键值 | 用户 ↔ 用户详情（身份证信息） |
| **一对多** | 在“多”的一方（表 B）创建一个字段，存储“一”的一方（表 A）的主键值 | 班级（一）↔ 学生（多）：学生表的 `cls_id` 存班级表的 `id` |
| **多对多** | **新建第三张表 C**，只有两个字段，一个存 A 的主键值，一个存 B 的主键值 | 学生 ↔ 课程：新建“选课表”（`student_id`、`course_id`） |

> **想一想**：举些例子，满足一对一、一对多、多对多的对应关系？例如：丈夫—妻子（一对一）、部门—员工（一对多）、老师—学生、作者—书籍（多对多）。

```sql
-- 多对多示例：学生选课的关系表（中间表）
create table student_course(
    -- 学生编号：对应 students.id
    student_id int unsigned,
    -- 课程编号：对应 courses.id
    course_id int unsigned,
    -- 联合主键：同一个学生不能重复选同一门课
    primary key(student_id, course_id)
);
```

### 7.4 外键的使用

> 以“班级（一）对学生（多）”为例，在学生表的 `cls_id` 上添加外键，引用班级表的 `id`。

```sql
-- 给 students 表的 cls_id 字段添加外键约束，引用 classes 表的 id 字段
-- 注意：两个字段的类型必须一致（这里都是 int unsigned），被引用的字段必须是主键或有索引，表引擎需要是 InnoDB
-- 如果表中已有数据，已有的 cls_id 必须都能在 classes.id 中找到，否则添加失败
alter table students add foreign key (cls_id) references classes(id);
-- 插入时 cls_id 必须在 classes 中存在，否则报错（Cannot add or update a child row）
insert into students(name,birth,cls_id) values('张三','2020-1-9',1);
```

> **提醒**：外键会拦截不合法的数据，但也有代价：删除或修改被引用的班级时，如果还有学生引用它，也会失败（需先处理学生）。这是外键“降低性能、增加耦合”的体现，所以实际开发中多由程序逻辑层保证。

### 7.5 设计示例：班级表与学生表

> 设计两张表：
>
> - **班级表 `classes`**：`id`、`name`、`isdelete`
> - **学生表 `students`**：`id`、`name`、`birthday`、`gender`、`clsid`、`isdelete`
>
> 其中 `clsid` 体现了班级与学生的**一对多**关系，`isdelete` 用于**逻辑删除**。

---

## 八、查询准备

> 从本章开始学习**查询（DQL）**，这是 SQL 中最重要、最复杂的部分。先创建测试库、测试表并准备数据，后面所有查询示例都基于它们。

### 8.1 创建数据库与数据表

```sql
-- 创建数据库
create database python_test_1 charset=utf8;
-- 使用数据库
use python_test_1;

-- 创建 students 表
create table students(
    -- 主键，自动增长
    id int unsigned primary key auto_increment not null,
    -- 姓名
    name varchar(20) default '',
    -- 年龄
    age tinyint unsigned default 0,
    -- 身高（cm）
    height decimal(5,2),
    -- 性别：枚举，默认 '保密'；序号 1=男，2=女，3=中性，4=保密
    gender enum('男','女','中性','保密') default '保密',
    -- 所属班级编号
    cls_id int unsigned default 0,
    -- 逻辑删除标记：bit 类型，0=未删除，1=已删除
    is_delete bit default 0
);

-- 创建 classes 表
create table classes (
    id int unsigned auto_increment primary key not null,
    -- 班级名称，不允许为空
    name varchar(30) not null
);
```

### 8.2 准备数据

```sql
-- 向 students 表中插入数据（gender 用枚举序号：1=男 2=女 3=中性 4=保密）
insert into students(name,age,height,gender,cls_id,is_delete) values
('小明', 18, 180.00, 2, 1, 0),
('小月月', 18, 180.00, 2, 2, 1),
('彭于晏', 29, 185.00, 1, 1, 0),
('刘德华', 59, 175.00, 1, 2, 1),
('黄蓉', 38, 160.00, 2, 1, 0),
('凤姐', 28, 150.00, 4, 2, 1),
('王祖贤', 18, 172.00, 2, 1, 1),
('周杰伦', 36, NULL, 1, 1, 0),
('程坤', 27, 181.00, 1, 2, 0),
('刘亦菲', 25, 166.00, 2, 2, 0),
('金星', 33, 162.00, 3, 3, 1),
('静香', 12, 180.00, 2, 4, 0),
('郭靖', 12, 170.00, 1, 4, 0),
('周杰', 34, 176.00, 2, 5, 0);

-- 向 classes 表中插入数据
insert into classes values (0, 'python_01期'), (0, 'python_02期');
```

> **数据概览**（后面的示例结果都基于它）：

| id | name | age | height | gender | cls_id | is_delete |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 小明 | 18 | 180.00 | 女 | 1 | 0 |
| 2 | 小月月 | 18 | 180.00 | 女 | 2 | 1 |
| 3 | 彭于晏 | 29 | 185.00 | 男 | 1 | 0 |
| 4 | 刘德华 | 59 | 175.00 | 男 | 2 | 1 |
| 5 | 黄蓉 | 38 | 160.00 | 女 | 1 | 0 |
| 6 | 凤姐 | 28 | 150.00 | 保密 | 2 | 1 |
| 7 | 王祖贤 | 18 | 172.00 | 女 | 1 | 1 |
| 8 | 周杰伦 | 36 | NULL | 男 | 1 | 0 |
| 9 | 程坤 | 27 | 181.00 | 男 | 2 | 0 |
| 10 | 刘亦菲 | 25 | 166.00 | 女 | 2 | 0 |
| 11 | 金星 | 33 | 162.00 | 中性 | 3 | 1 |
| 12 | 静香 | 12 | 180.00 | 女 | 4 | 0 |
| 13 | 郭靖 | 12 | 170.00 | 男 | 4 | 0 |
| 14 | 周杰 | 34 | 176.00 | 女 | 5 | 0 |

> **注意**：`classes` 表只有 2 个班级（id 为 1、2），而学生的 `cls_id` 中有 3、4、5，它们在班级表中找不到对应记录，这正是后面“连接查询”区分内、左、右连接的关键数据。

---

## 九、基础查询

### 9.1 查询所有字段与指定字段

```sql
-- 查询所有字段：select * from 表名;
select * from students;
-- 查询指定字段：select 列1,列2,... from 表名;
select name from students;
```

### 9.2 使用 `as` 起别名

> `as` 可以给**字段**或**表**起别名（`as` 可以省略，但建议写上，更清晰）。别名常用于：让结果列名更易读；表名太长时简写；同一张表自连接时区分（见第十七章）。

```sql
-- 给字段起别名：结果集的列标题变为 序号、名字、性别
select id as 序号, name as 名字, gender as 性别 from students;

-- 如果是单表查询，可以省略表名
select id, name, gender from students;
-- 也可以用 表名.字段名 的形式（多表查询时用来区分同名字段）
select students.id, students.name, students.gender from students;
-- 通过 as 给表起别名：之后用别名 s 引用该表
select s.id, s.name, s.gender from students as s;
```

### 9.3 消除重复行（`distinct`）

> 在 `select` 后面、列前使用 `distinct` 可以消除重复的行。
>
> 语法：`select distinct 列1,... from 表名;`
>
> **注意**：`distinct` 作用于**后面所有列的组合**，即多个列时，是这些列的组合重复才去重。

```sql
-- 查询学生表中出现过的性别（去重，只显示 男/女/中性/保密 四种）
select distinct gender from students;
-- 多列去重：(gender, cls_id) 组合相同的才会被去重
select distinct gender, cls_id from students;
```

---

## 十、条件查询

> 使用 `where` 子句对表中的数据**筛选**，结果为 `true` 的行才会出现在结果集中。
>
> 语法：`select * from 表名 where 条件;`
>
> `where` 后面支持多种运算符：比较运算符、逻辑运算符、模糊查询、范围查询、空判断。

```sql
-- 查询 id 为 1 的学生
select * from students where id=1;
```

### 10.1 比较运算符

| 运算符 | 含义 | 备注 |
| --- | --- | --- |
| `=` | 等于 | SQL 中判断相等用**一个**等号，不是 `==` |
| `>` | 大于 | |
| `>=` | 大于等于 | |
| `<` | 小于 | |
| `<=` | 小于等于 | |
| `!=` 或 `<>` | 不等于 | 讲义标注“低效率操作，不要使用”：不等于通常无法利用索引，大表上应尽量避免 |

```sql
-- 例 1：查询编号大于 3 的学生
select * from students where id > 3;
-- 例 2：查询编号不大于 4 的学生
select * from students where id <= 4;
-- 例 3：查询姓名不是“黄蓉”的学生
select * from students where name != '黄蓉';
-- 例 4：查询没被删除的学生（is_delete 是 bit 类型，0 表示未删除）
select * from students where is_delete=0;
```

### 10.2 逻辑运算符

| 运算符 | 含义 | 备注 |
| --- | --- | --- |
| `and` | 并且（两边条件都为真才为真） | |
| `or` | 或者（任意一边为真就为真） | |
| `not` | 取反 | 讲义标注“这个不好，不要用”：可读性差，通常可以改写为 `!=`、`not in`、`is not null` 等 |

```sql
-- 例 5：查询编号大于 3 的女同学
select * from students where id > 3 and gender=2;
-- 例 6：查询编号小于 4 或没被删除的学生
select * from students where id < 4 or is_delete=0;
```

### 10.3 模糊查询（`like`）

> `like` 用于**字符串**的模糊匹配，有两个通配符：
>
> - `%`：表示**零个或任意多个**任意字符
> - `_`：表示**一个**任意字符

```sql
-- 例 7：查询姓黄的学生（黄 + 任意多个字符）
select * from students where name like '黄%';
-- 例 8：查询姓黄并且“名”是一个字的学生（黄 + 恰好 1 个字符）
select * from students where name like '黄_';
-- 例 9：查询姓黄或名字以“靖”结尾的学生
select * from students where name like '黄%' or name like '%靖';
```

> **性能提示（补充）**：`like '黄%'`（前缀匹配）可以使用索引；`like '%靖'`（以通配符开头）无法使用索引，会全表扫描。

### 10.4 范围查询

| 写法 | 含义 |
| --- | --- |
| `in (值1,值2,...)` | 在一个**非连续**的范围内（列表中任意一个） |
| `between 值1 and 值2` | 在一个**连续**的范围内，**闭区间**（包含两端），值1 要小于等于值2 |

```sql
-- 例 10：查询编号是 1 或 3 或 8 的学生
select * from students where id in (1,3,8);
-- 例 11：查询编号为 3 至 8 的学生（包含 3 和 8）
select * from students where id between 3 and 8;
-- 例 12：查询编号是 3 至 8 的男生
select * from students where id between 3 and 8 and gender=1;
```

### 10.5 空判断

> **注意**：`NULL` 与 `''`（空字符串）是**不同的**！`NULL` 表示“未知 / 没有值”，`''` 是一个长度为 0 的字符串。
>
> 判断是否为 `NULL` **不能**用 `= null`（结果永远不为真），必须用：
>
> - `is null`：判空
> - `is not null`：判非空

```sql
-- 例 13：查询没有填写身高的学生（结果：周杰伦）
select * from students where height is null;
-- 例 14：查询填写了身高的学生
select * from students where height is not null;
-- 例 15：查询填写了身高的男生
select * from students where height is not null and gender=1;
```

### 10.6 运算优先级

> 优先级由高到低依次为：**小括号 → `not` → 比较运算符 → 逻辑运算符**。
>
> 在逻辑运算符中，**`and` 比 `or` 先运算**。如果同时出现，并希望先算 `or`，需要结合小括号 `()` 使用。

```sql
-- 没有括号：先算 and，即 “(性别=男 且 id>10) 或 (is_delete=0)”
select * from students where gender=1 and id>10 or is_delete=0;
-- 加括号：先算 or，即 “性别=男 且 (id>10 或 is_delete=0)”，两条语句结果不同
select * from students where gender=1 and (id>10 or is_delete=0);
```

---

## 十一、排序

> 为了方便查看数据，可以对数据进行排序。
>
> 语法：`select * from 表名 order by 列1 asc|desc [,列2 asc|desc,...]`
>
> **说明**：
>
> - 将行数据按照**列 1** 进行排序；如果某些行**列 1 的值相同**，则按照**列 2** 排序，以此类推
> - 默认按照列值**从小到大**排列（即默认 `asc`）
> - `asc`：从小到大，**升序**
> - `desc`：从大到小，**降序**

```sql
-- 例 1：查询未删除男生信息，按学号降序
select * from students where gender=1 and is_delete=0 order by id desc;
-- 例 2：查询未删除学生信息，按名称升序（asc 可省略；中文按字符集的排序规则排序，不是拼音顺序）
select * from students where is_delete=0 order by name;
-- 例 3：显示所有学生信息，先按年龄从大到小排序，年龄相同时按身高从高到矮排序
select * from students
order by age desc, height desc;
```

> **注意**：`order by` 要写在 `where` **之后**。`NULL` 值在升序时排在最前，降序时排在最后。

---

## 十二、聚合函数

> 为了快速得到**统计数据**，经常会用到下面 5 个聚合函数。聚合函数对**一组行**进行计算，返回**一个值**。
>
> ---

| 函数 | 含义 | 说明 |
| --- | --- | --- |
| `count(*)` | 总行数 | 括号中写 `*` 与写列名（非空列）结果相同；`count(列名)` **不统计该列为 `NULL` 的行** |
| `max(列)` | 最大值 | 求此列的最大值 |
| `min(列)` | 最小值 | 求此列的最小值 |
| `sum(列)` | 求和 | 求此列的和 |
| `avg(列)` | 平均值 | 求此列的平均值 |

> ---
>
> **补充**：聚合函数会**忽略 `NULL`**（`count(*)` 除外），如 `avg(height)` 不会把 `周杰伦` 的空身高当成 0。

```sql
-- 例 1：查询学生总数（结果：14）
select count(*) from students;
-- 例 2：查询女生的编号最大值（结果：13）
select max(id) from students where gender=2;
-- 例 3：查询未删除的学生最小编号（结果：1）
select min(id) from students where is_delete=0;
-- 例 4：查询男生的总年龄（29+59+36+27+12=163）
select sum(age) from students where gender=1;
-- 平均年龄：总年龄 / 人数（结果：32.6，等价于 avg(age)）
select sum(age)/count(*) from students where gender=1;
-- 例 5：查询未删除女生的编号平均值（id 为 1、5、10、12、13，平均值 8.2）
select avg(id) from students where is_delete=0 and gender=2;
```

---

## 十三、分组

### 13.1 `group by`

> **含义**：将查询结果按照 **1 个或多个字段**进行分组，**字段值相同的为一组**。`group by` 可用于单个字段分组，也可用于多个字段分组。

```sql
-- 按性别分组：gender 共有 4 个不同的值，所以分为 4 组
select gender from students group by gender;
```

```
+--------+
| gender |
+--------+
| 男     |
| 女     |
| 中性   |
| 保密   |
+--------+
```

> 当 `group by` **单独使用**时，只显示出每组的第一条记录（去重效果），实际意义不大。**分组通常要配合聚合函数、`group_concat()`、`having`、`with rollup` 使用**。
>
> **重要规则**：`select` 后面出现的列，要么出现在 `group by` 中，要么放在聚合函数里；否则在默认的 `ONLY_FULL_GROUP_BY` 模式下（5.7 及 8.0 默认开启）会报错。

### 13.2 `group by` + `group_concat()`

> `group_concat(字段名)` 可以作为一个输出字段使用，表示**分组之后，把每一组的某字段的值拼接成一个字符串**（`concat` 是“构造合并”的含义，默认用逗号分隔）。

```sql
-- 每个性别分组下，有哪些人的姓名
select gender, group_concat(name) from students group by gender;
```

```
+--------+-----------------------------------------------------------+
| gender | group_concat(name)                                        |
+--------+-----------------------------------------------------------+
| 男     | 彭于晏,刘德华,周杰伦,程坤,郭靖                            |
| 女     | 小明,小月月,黄蓉,王祖贤,刘亦菲,静香,周杰                  |
| 中性   | 金星                                                      |
| 保密   | 凤姐                                                      |
+--------+-----------------------------------------------------------+
```

```sql
-- 每个性别分组下，有哪些学生的 id
select gender, group_concat(id) from students group by gender;
```

```
+--------+------------------+
| gender | group_concat(id) |
+--------+------------------+
| 男     | 3,4,8,9,14       |
| 女     | 1,2,5,7,10,12,13 |
| 中性   | 11               |
| 保密   | 6                |
+--------+------------------+
```

### 13.3 `group by` + 聚合函数

> 既然可以统计出每个分组的某字段的值的集合，那么也可以用**聚合函数**对每组的值做计算：每个分组算一个结果。

```sql
-- 分别统计每种性别的平均年龄
select gender, avg(age) from students group by gender;
```

```
+--------+----------+
| gender | avg(age) |
+--------+----------+
| 男     |  32.6000 |
| 女     |  23.2857 |
| 中性   |  33.0000 |
| 保密   |  28.0000 |
+--------+----------+
```

```sql
-- 分别统计每种性别的人数
select gender, count(*) from students group by gender;
```

```
+--------+----------+
| gender | count(*) |
+--------+----------+
| 男     |        5 |
| 女     |        7 |
| 中性   |        1 |
| 保密   |        1 |
+--------+----------+
```

### 13.4 `group by` + `having`

> `having 条件表达式`：用来**对分组之后的结果**再指定条件。
>
> **`where` 与 `having` 的区别**：
>
> ---

| 对比 | `where` | `having` |
| --- | --- | --- |
| 作用对象 | **原始表**中的行（分组之前过滤） | **分组之后**的结果（分组后过滤） |
| 能否使用聚合函数 | 不能 | **能**（如 `count(*) > 2`） |
| 是否依赖 `group by` | 否 | 一般和 `group by` 配合使用 |

> ---

```sql
-- 查询人数大于 2 的性别分组（中性、保密各只有 1 人，被过滤掉）
select gender, count(*) from students group by gender having count(*) > 2;
```

```
+--------+----------+
| gender | count(*) |
+--------+----------+
| 男     |        5 |
| 女     |        7 |
+--------+----------+
```

```sql
-- where 和 having 同时使用：先用 where 过滤出未删除的学生，再分组，再用 having 过滤人数
select gender, count(*) from students where is_delete=0
group by gender having count(*) > 2;
```

### 13.5 `group by` + `with rollup`

> `with rollup` 的作用是：在最后**新增一行**，用来记录当前列里**所有记录的汇总**（该行分组列的值为 `NULL`）。如果使用 `having`，要放在 `with rollup` 之后。

```sql
-- 按性别统计人数，并在最后一行汇总总人数
select gender, count(*) from students group by gender with rollup;
```

```
+--------+----------+
| gender | count(*) |
+--------+----------+
| 男     |        5 |
| 女     |        7 |
| 中性   |        1 |
| 保密   |        1 |
| NULL   |       14 |
+--------+----------+
```

```sql
-- 与 group_concat 结合：最后一行把所有年龄汇总在一起
select gender, group_concat(age) from students group by gender with rollup;
```

```
+--------+-------------------------------------------+
| gender | group_concat(age)                         |
+--------+-------------------------------------------+
| 男     | 29,59,36,27,12                            |
| 女     | 18,18,38,18,25,12,34                      |
| 中性   | 33                                        |
| 保密   | 28                                        |
| NULL   | 29,59,36,27,12,18,18,38,18,25,12,34,33,28 |
+--------+-------------------------------------------+
```

---

## 十四、窗口函数（MySQL 8.0 新增）

> **窗口函数**可以在**不合并行**的前提下，对每一行基于“一个窗口（一组相关的行）”进行计算，典型用途是**排名**。与 `group by` 的区别：`group by` 会把每组合并为一行，窗口函数**保留所有原始行**，并多出一列计算结果。
>
> 语法：`函数() over (partition by 分组列 order by 排序列)`
>
> - `partition by`：把数据按列分成若干“窗口”（可省略，省略则整张表为一个窗口）
> - `order by`：窗口内的排序规则

> **三个排名函数的区别**（以“同年龄并列”为例）：
>
> ---

| 函数 | 含义 | 遇到并列时 |
| --- | --- | --- |
| `rank()` | 排名，**并列后会跳号** | 如 1, 2, 2, 4 |
| `dense_rank()` | 密集排名，**并列后不跳号** | 如 1, 2, 2, 3 |
| `row_number()` | 行号，**不并列**，依次编号 | 如 1, 2, 3, 4（并列行的先后顺序不确定） |

> ---

```sql
-- 按班级分组，在每个班级内按年龄从大到小排名：
-- rank1：rank() 排名；dese_rank：dense_rank() 排名；row_num：row_number() 行号
select *,
    rank() over (partition by cls_id order by age desc) as rank1,
    dense_rank() over (partition by cls_id order by age desc) as dese_rank,
    row_number() over (partition by cls_id order by age desc) as row_num
from students;

-- 不分组，对全体学生按年龄从大到小排名
select *,
    rank() over (order by age desc) as rank1,
    dense_rank() over (order by age desc) as dese_rank,
    row_number() over (order by age desc) as row_num
from students;
```

> **结果解读**：对全体学生按年龄降序，年龄 18 岁有 3 人（小明、小月月、王祖贤）并列。前面比他们大的有 9 人（59、38、36、34、33、29、28、27、25 岁）。按 `rank()`，这 3 人均为第 10 名，下一个 12 岁的是第 13 名（跳过 11、12）；按 `dense_rank()`，这 3 人均为第 10 名，下一个 12 岁是第 11 名（不跳号）；按 `row_number()`，则是 10、11、12、13……依次编号（并列行之间的先后顺序不确定）。
>
> 参考：[MySQL 官方窗口函数文档](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html)。

---

## 十五、分页（获取部分行）

> 当数据量过大时，在一页中查看数据非常麻烦，所以需要**分页**显示。
>
> 语法：`select * from 表名 limit start, count`
>
> **说明**：从 `start` 开始（**从第 0 行开始计数**，最上面一行是第 0 行），获取 `count` 条数据。
>
> 也可写作 `limit count offset start`；如果只写一个数 `limit count`，等价于 `limit 0, count`。

```sql
-- 例 1：查询前 3 行男生信息（从第 0 行开始，取 3 条）
select * from students where gender=1 limit 0,3;
-- 取第 4~6 行：从第 3 行开始，取 3 条
select * from students limit 3,3;
```

### 15.1 分页公式

> 已知：**每页显示 `m` 条数据，当前显示第 `n` 页**。
>
> **求总页数**（此逻辑在后面的 Python 程序中实现）：
>
> 1. 查询总条数 `p1`
> 2. 使用 `p1` 除以 `m` 得到 `p2`
> 3. 如果整除，则 `p2` 就是总页数；如果不整除，则 `p2 + 1` 为总页数（即**向上取整**）
>
> **求第 `n` 页的数据**：`limit (n-1)*m, m`

```sql
-- 总页数：每页 5 条，用 ceiling() 向上取整（students 有 14 条，14/5=2.8，向上取整为 3）
select ceiling(count(*)/5) from students;
-- 第 n 页的数据（每页 m 条）：讲义写作 limit (n-1)*m,m
-- 注意：limit 后面只能写数字，不能直接写表达式，(n-1)*m 要由程序先算好
-- 例：每页 5 条，第 3 页，则 (3-1)*5=10，即 limit 10,5
select * from students where is_delete=0 limit 10,5;
```

> **注意**：分页时最好配合 `order by` 指定排序，否则不保证每次返回的行顺序一致，可能出现翻页重复或遗漏。

---

## 十六、连接查询

> 当查询结果的**列来源于多张表**时，需要将多张表**连接**成一个大的数据集，再选择合适的列返回。
>
> MySQL 支持三种类型的连接查询：

| 连接类型 | 关键字 | 结果 |
| --- | --- | --- |
| **内连接** | `inner join` | 只返回**两个表匹配到**的数据 |
| **左连接** | `left join` | 两个表匹配到的数据 + **左表特有**的数据（右表中不存在的部分用 `NULL` 填充） |
| **右连接** | `right join` | 两个表匹配到的数据 + **右表特有**的数据（左表中不存在的部分用 `NULL` 填充） |

> **语法**：`select * from 表1 inner|left|right join 表2 on 表1.列 = 表2.列`
>
> - `on` 后面是**连接条件**（两张表通过哪一列对应起来）
> - 多表查询时，常用 `as` 给表起别名，目的是编写简单；同名列要用 `表名.列名`（或 `别名.列名`）区分
> - `inner` 可以省略，直接写 `join`
>
> 记忆方式：以**左表**（写在 `join` 前面的）为主 → `left join`，左表的所有行都保留；以**右表**为主 → `right join`。
>
> MySQL **不支持** `full join`（全连接），如果需要可以用 `left join union right join` 实现。

```sql
-- 例 1：内连接，查询学生与班级的对应数据
-- 只有 cls_id 为 1、2 的学生才能在 classes 中找到班级，所以结果是 10 行
select * from students inner join classes on students.cls_id = classes.id;

-- 例 2：左连接（以学生表为主）：14 个学生全部保留，cls_id 为 3、4、5 的学生，班级信息为 NULL
select * from students as s left join classes as c on s.cls_id = c.id;

-- 例 3：右连接（以班级表为主）：班级表的所有班级都保留，没有学生的班级学生信息为 NULL
-- 本例中两个班级都有学生，所以结果也是 10 行
select * from students as s right join classes as c on s.cls_id = c.id;

-- 例 4：查询学生姓名及班级名称（只选部分列，两个表都有 name 列，必须用别名区分）
select s.name, c.name from students as s inner join classes as c on s.cls_id = c.id;
```

> **三种连接的结果对比**（基于第八章的数据）：

| 连接 | 结果行数 | 说明 |
| --- | --- | --- |
| `inner join` | 10 | 只有 `cls_id` 为 1 或 2 的 10 名学生 |
| `left join`（学生为左表） | 14 | 10 名匹配的学生 + `金星`、`静香`、`郭靖`、`周杰` 4 名没有班级的学生（班级列为 `NULL`） |
| `right join`（班级为右表） | 10 | 两个班级都有学生，没有“只在班级表中存在”的数据 |

---

## 十七、自关联

> 以**省市区**数据为例。最直观的设计是两张表：
>
> - 省信息表 `provinces`：`id`、`ptitle`
> - 市信息表 `citys`：`id`、`ctitle`、`proid`（`proid` 表示城市所属的省，对应 `provinces` 表的 `id`）
>
> **问题**：能不能把两张表合成一张表？
>
> **思考**：观察发现，`citys` 表只比 `provinces` 表多一个 `proid` 列，其他列的类型都一样。**存储的都是地区信息**，每种信息的数据量有限，没必要新增表；将来还可能存储区县、乡镇信息，每增加一级都新增一张表，开销太大。
>
> **答案**：定义一张 `areas` 表，结构如下：
>
> - `aid`：编号（主键）
> - `atitle`：地区名称
> - `pid`：上级地区的编号
>
> **说明**：
>
> - 省没有所属的上级，所以 `pid` 填写 `NULL`
> - 城市的 `pid` 填写所属省对应的 `aid`
> - 这就是**自关联**：表中的某一列，关联了**这个表中的另外一列**，但它们的**业务逻辑含义不同**——城市的 `pid` 引用的是省的 `aid`
> - 表结构不变，还可以继续添加区县、乡镇街道、村社区等信息

```sql
-- 创建 areas 表：aid 主键，atitle 名称，pid 上级编号（省的 pid 为 NULL）
create table areas(
    aid int primary key,
    atitle varchar(20),
    pid int
);
```

```sql
-- 从 SQL 脚本文件中导入数据（areas.sql 需要在当前目录或写完整路径）
source areas.sql;
-- 查询一共有多少个省（省的 pid 为 NULL）
select count(*) from areas where pid is null;
```

> **自连接的写法**：把同一张表当成两张表，用**不同的别名**区分，再用连接条件将“下级”的 `pid` 与“上级”的 `aid` 对应起来。

```sql
-- 例 1：查询省的名称为“山西省”的所有城市
-- c 代表“城市”（下级），p 代表“省”（上级）；连接条件：城市的 pid = 省的 aid
select c.* from areas as c
inner join areas as p on c.pid = p.aid
where p.atitle = '山西省';

-- 例 2：查询市的名称为“广州市”的所有区县
-- dis 代表区县，city 代表市
select dis.* from areas as dis
inner join areas as city on city.aid = dis.pid
where city.atitle = '广州市';
```

---

## 十八、子查询

> **子查询**：在一个 `select` 语句中，**嵌入了另外一个 `select` 语句**，被嵌入的 `select` 语句称为**子查询语句**；外面的第一条 `select` 语句称为**主查询**。
>
> **主查询和子查询的关系（规则）**：
>
> - 子查询是**嵌入到主查询中**的
> - 子查询是**辅助主查询**的：要么充当**条件**，要么充当**数据源**
> - 子查询是可以**独立存在**的语句，是一条完整的 `select` 语句
>
> **子查询的分类**：
>
> ---

| 分类 | 返回结果 | 常配合的运算符 |
| --- | --- | --- |
| 标量子查询 | 一个数据（**一行一列**） | `=`、`>`、`<` 等比较运算符 |
| 列子查询 | 一列（**一列多行**） | `in`（范围） |
| 行子查询 | 一行（**一行多列**） | 行元素 `(列1,列2) = (子查询)` |

> ---

### 18.1 标量子查询

> 需求：查询身高大于平均身高的学生。拆成两步：①求平均身高；②查询大于该值的学生。用子查询一步完成：

```sql
-- 子查询 (select avg(height) from students) 返回一个数（一行一列），充当主查询的条件
select * from students where height > (select avg(height) from students);
```

### 18.2 列子查询

> 需求：查询**还有学生在读**的所有班级的名字。拆成两步：①找出学生表中所有的班级 id；②找出班级表中对应的名字。

```sql
-- 子查询返回一列多行（所有出现过的 cls_id），主查询用 in 判断班级 id 是否在其中
select name from classes where id in (select cls_id from students);
```

### 18.3 行子查询

> 需求：查找班级中**年龄最大、身高最高**的学生。
>
> **行元素**：将多个字段合成一个**行元素**（如 `(height, age)`），在行子查询中使用。

```sql
-- 子查询返回一行两列 (最大身高, 最大年龄)，主查询用行元素 (height, age) 与它比较
select * from students
where (height, age) = (select max(height), max(age) from students);
```

> **注意**：此例中，最高身高 185（彭于晏）和最大年龄 59（刘德华）**不是同一个人**，所以没有学生同时满足 `height=185 and age=59`，这条语句的结果是**空**。行子查询要求“同一行同时满足”，使用时要注意业务含义。

### 18.4 子查询使用要点

> - **`in` 范围**：格式 `主查询 where 条件 in (列子查询)`
> - 子查询用在 `=`、`>` 等比较运算符后面时，**只能返回一行一列**，返回多行会报错 `Subquery returns more than 1 row`
> - 子查询要写在**小括号**里
> - 能用**连接查询**实现的需求，通常连接比子查询**效率更高**，可读性好的场合再选子查询

---

## 十九、SELECT 完整语法与执行顺序

### 19.1 完整语法

> 查询的完整格式（看起来复杂，其实很简单，**实际使用中只是其中某些部分的组合，而不是全部**）：

```sql
-- [ ] 表示可选部分
SELECT [DISTINCT] select_expr [, select_expr, ...]
[FROM tb_name]
[WHERE 条件判断]
[GROUP BY {col_name | position} [ASC | DESC], ...]
[HAVING 条件判断]
[ORDER BY {col_name | expr | position} [ASC | DESC], ...]
[LIMIT {[offset,] row_count | row_count OFFSET offset}];
```

```sql
-- 完整的 select 语句示例骨架
select distinct *
from 表名
where ....
group by ... having ...
order by ...
limit start, count
```

### 19.2 执行顺序

> 各个子句的**书写顺序**与**执行顺序**不同：书写时 `select` 在最前，但它是在分组、`having` 之后才执行的。**正确的执行顺序**为：

| 顺序 | 子句 | 作用 |
| --- | --- | --- |
| 1 | `from 表名` | 确定数据来源（多表时先完成连接） |
| 2 | `where ....` | 对**原始行**过滤 |
| 3 | `group by ...` | 分组 |
| 4 | `having ...` | 对**分组结果**过滤 |
| 5 | `select [distinct] ...` | 计算要输出的列、起别名、去重 |
| 6 | `order by ...` | 排序 |
| 7 | `limit start,count` | 截取部分行（分页） |

> **说明**：PDF 讲义中把 `select distinct *` 列在 `having` 之前，严格来说 `select`（含 `distinct`）是在 `having` **之后**才执行的。理解这一点可以解释一些现象，例如：`where` 中不能使用 `select` 中定义的聚合结果；`order by` 中可以使用别名（因为它在 `select` 之后执行）。

---

## 二十、视图

### 20.1 为什么需要视图

> **问题**：对于复杂的查询，往往是由多个数据表进行**关联查询**而得到的。如果数据库因为需求等原因发生了改变，为了保证查询出来的数据与之前相同，就需要在多个地方进行修改，维护起来非常麻烦。
>
> **解决办法**：定义**视图**。

### 20.2 视图是什么

> 通俗地讲，**视图就是一条 `select` 语句执行后返回的结果集**。所以创建视图的时候，主要工作就落在创建这条 SQL 查询语句上。
>
> - 视图是对若干张基本表的引用，是一张**虚表**，是查询语句执行的结果，**不存储具体的数据**（基本表的数据发生了改变，视图也会跟着改变）
> - 方便操作，特别是查询操作，**减少复杂的 SQL 语句，增强可读性**

### 20.3 视图的使用

> **命名建议**：视图名以 `v_` 开头，便于和普通表区分。

```sql
-- 1) 定义视图：create view 视图名称 as select 语句;
-- 例：把“学生姓名 + 班级名称”的连接查询定义为视图 v_stu_class
create view v_stu_class as
select s.id as id, s.name as sname, c.name as cname
from students as s
left join classes as c on s.cls_id = c.id;

-- 2) 查看视图：show tables 会把所有的视图也一起列出来
show tables;
-- 3) 使用视图：视图的用途就是查询，用法和普通表一样
select * from v_stu_class;
select * from v_stu_class where cname = 'python_01期';
-- 4) 修改视图：create or replace view 可以在视图已存在时直接替换定义
create or replace view v_stu_class as
select s.id as id, s.name as sname, s.age as age, c.name as cname
from students as s
left join classes as c on s.cls_id = c.id;
-- 5) 查看视图的创建语句
show create view v_stu_class;
-- 6) 删除视图：drop view 视图名称;
drop view v_stu_class;
```

> **说明**：
>
> - 讲义示例使用的是 `jing_dong` 库：`create view goods_view as (select goods.id as id, goods.name as name, goods_cates.name as type from goods left outer join goods_cates on goods_cates.id = goods.cate_id);`，思路与上面相同
> - 简单的视图（单表、不含聚合、分组、`distinct` 等）通常可以通过视图 `insert`/`update`/`delete`，会作用到基本表；复杂视图（连接、聚合等）一般**不可更新**，所以视图主要用于**查询**

### 20.4 视图的作用

| 作用 | 说明 |
| --- | --- |
| 提高重用性 | 就像一个**函数**，复杂查询写一次，到处使用 |
| 隔离表结构变化 | 对数据库**重构**（如拆表、改列名）时，只需修改视图定义，**不影响程序的运行** |
| 提高安全性 | 可以只暴露部分列、部分行给不同的用户（配合权限，只授予视图的查询权限） |
| 让数据更加清晰 | 把想要的数据整理成一张逻辑上的表，便于理解 |

---

## 二十一、事务

### 21.1 为什么要有事务

> 事务广泛地运用于**订单系统、银行系统**等多种场景。
>
> **例**：A 用户和 B 用户是银行的储户，现在 A 要给 B 转账 500 元，需要做以下几件事：
>
> 1. 检查 A 的账户余额 > 500 元
> 2. A 账户中扣除 500 元
> 3. B 账户中增加 500 元
>
> 正常流程走下来，A 扣了 500，B 加了 500，皆大欢喜。**但如果 A 账户扣了钱之后，系统出故障了呢？** A 白白损失了 500，而 B 也没有收到本该属于他的 500。
>
> 以上案例中，隐藏着一个前提：**A 扣钱和 B 加钱，要么同时成功，要么同时失败**。事务的需求就在于此。
>
> **事务**：是一个**操作序列**，这些操作**要么都执行，要么都不执行**，它是一个**不可分割的工作单位**。事务是数据库**维护数据一致性的单位**，在每个事务结束时，都能保持数据一致性。

> **银行转账的经典例子**（出自《高性能 MySQL》第三版）：
>
> 假如银行的数据库有两张表：支票表 `checking` 和储蓄表 `savings`。现在要从用户 Jane 的支票账户转移 200 美元到她的储蓄账户，至少需要三个步骤：
>
> 1. 检查支票账户的余额高于或者等于 200 美元
> 2. 从支票账户余额中减去 200 美元
> 3. 在储蓄账户余额中增加 200 美元
>
> 上述三个步骤的操作必须打包在**一个事务**中，任何一个步骤失败，则必须**回滚**所有的步骤。

```sql
-- 开始一个事务
start transaction;
-- 步骤 1：检查支票账户余额（应该 >= 200）
select balance from checking where customer_id = 10233276;
-- 步骤 2：从支票账户扣 200
update checking set balance = balance - 200.00 where customer_id = 10233276;
-- 步骤 3：给储蓄账户加 200
update savings set balance = balance + 200.00 where customer_id = 10233276;
-- 提交事务：所有修改持久保存；如果中间出错，应该执行 rollback; 撤销所有修改
commit;
```

### 21.2 事务的四大特性（ACID）

| 特性 | 英文 | 含义 | 以转账为例 |
| --- | --- | --- | --- |
| **原子性** | Atomicity | 一个事务是**不可分割的最小工作单元**，所有操作**要么全部提交成功，要么全部失败回滚**，不可能只执行一部分 | 扣钱和加钱要么都成功，要么都不发生 |
| **一致性** | Consistency | 数据库总是从**一个一致性的状态转换到另一个一致性的状态** | 即使执行到一半系统崩溃，也不会凭空少了 200 美元，因为事务没提交，修改不会保存 |
| **隔离性** | Isolation | 一个事务所做的修改在**最终提交以前，对其他事务是不可见的** | 扣了钱还没加钱时，另一个汇总程序看到的支票余额**并没有**被减去 200 |
| **持久性** | Durability | 一旦事务**提交**，其所做的修改会**永久保存**到数据库 | 提交后即使系统崩溃，修改的数据也不会丢失 |

### 21.3 事务命令

> **前提**：表的引擎类型必须是 **InnoDB** 才可以使用事务，这是 MySQL 表的**默认引擎**（`MyISAM` 引擎不支持事务）。
>
> 可以查看表的创建语句，末尾能看到 `ENGINE=InnoDB`。

```sql
-- 选择数据库（讲义用的 jing_dong 库，可换成自己的库）
use jing_dong;
-- 查看 goods 表的创建语句，确认引擎是 InnoDB
show create table goods;
```

| 命令 | 作用 |
| --- | --- |
| `begin;` 或 `start transaction;` | **开启事务**。开启后执行的修改命令，变更先维护在**缓存**中，不会真正写入物理表 |
| `commit;` | **提交事务**：将缓存中的数据变更维护到物理表中，修改生效 |
| `rollback;` | **回滚事务**：放弃缓存中变更的数据，回到事务开始前的状态 |

> **注意**：
>
> 1. **修改数据的命令会自动触发事务**，包括 `insert`、`update`、`delete`。MySQL 默认开启**自动提交**（`autocommit=1`），即不手动开启事务时，**每条** DML 语句执行完就自动提交一次
> 2. 在 SQL 语句中**手动开启事务**的原因是：可以进行**多次**数据的修改，如果成功就一起成功，否则一起回滚到之前的数据
> 3. `create`、`drop`、`alter` 等 DDL 语句会**隐式提交**当前事务，所以不能回滚；不要在事务中混用

```sql
-- 查看自动提交是否开启：1 表示开启，0 表示关闭
show variables like 'autocommit';
-- 临时关闭当前会话的自动提交（此后需要手动 commit 才生效），0 关闭，1 开启
set autocommit = 0;
```

### 21.4 回滚演示

> 为了演示效果，需要**打开两个终端窗口**，使用同一个数据库，操作同一张表（讲义用的是 `jing_dong` 库的 `goods_cates` 表）。

| 步骤 | 终端 1 | 终端 2 |
| --- | --- | --- |
| step1：连接 | `select * from goods_cates;`（记录现有数据） | — |
| step2：增加数据 | — | `begin;` 然后 `insert into goods_cates(name) values('小霸王游戏机');`，再 `select * from goods_cates;`，**此时能看到新增数据** |
| step3：查询 | `select * from goods_cates;`，**发现没有新增的数据**（事务未提交，对其他事务不可见，体现**隔离性**） | — |
| step4：回滚 | — | `rollback;` |
| step5：查询 | `select * from goods_cates;`，**仍然没有新增数据**（修改已被撤销） | `select * from goods_cates;` 同样没有 |

### 21.5 提交演示

| 步骤 | 终端 1 | 终端 2 |
| --- | --- | --- |
| step1：连接 | `select * from goods_cates;`（查询商品分类信息） | — |
| step2：增加数据 | — | `begin;` 然后 `insert into goods_cates(name) values('小霸王游戏机');`，再查询，**此时有新增数据** |
| step3：查询 | `select * from goods_cates;`，**发现没有新增的数据** | — |
| step4：提交 | — | `commit;` |
| step5：查询 | `select * from goods_cates;`，**发现有新增的数据**（事务已提交，修改持久保存且对其他事务可见） | — |

### 21.6 隔离级别（补充）

> 隔离性并不是“完全隔离”，为了兼顾性能，SQL 标准定义了**四种隔离级别**（《高性能 MySQL》中提到的“四种隔离级”）。隔离级别越高，数据越安全，但并发性能越低。

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 说明 |
| --- | --- | --- | --- | --- |
| `READ UNCOMMITTED`（读未提交） | 可能 | 可能 | 可能 | 能读到别的事务**未提交**的修改，几乎不用 |
| `READ COMMITTED`（读已提交） | 不会 | 可能 | 可能 | 只能读到已提交的数据；Oracle 等的默认级别 |
| `REPEATABLE READ`（可重复读） | 不会 | 不会 | 基本不会（InnoDB 通过间隙锁基本避免） | **InnoDB 的默认级别** |
| `SERIALIZABLE`（串行化） | 不会 | 不会 | 不会 | 事务串行执行，并发最差 |

> **名词解释**：
>
> - **脏读**：读到了别的事务**还没提交**的数据
> - **不可重复读**：同一个事务里，两次读同一行，因为别的事务**修改并提交**了，结果不一样
> - **幻读**：同一个事务里，两次按同样条件查询，因为别的事务**插入或删除并提交**了，行数不一样

```sql
-- 查看当前会话的隔离级别（8.0 用 transaction_isolation；5.7 用 tx_isolation）
select @@transaction_isolation;
-- 设置当前会话的隔离级别为“读已提交”
set session transaction isolation level read committed;
```

---

## 二十二、索引

### 22.1 为什么需要索引

> **思考**：在图书馆中是如何找到一本书的？——不会一本一本翻，而是先看**目录/分类索引**。
>
> 一般的应用系统，数据库的**读写比例在 10:1 左右**（即 10 次查询才有 1 次写操作），而且插入和更新操作很少出现性能问题；遇到最多、最容易出问题的还是一些**复杂的查询操作**，所以查询语句的优化显然是**重中之重**。
>
> 当数据库中数据量很大时，查找数据会变得很慢。**优化方案：索引**——就是**有序的数据结构**，常见有 **B+ 树索引**、**哈希索引**。

### 22.2 索引是什么

> **索引**是一种特殊的文件（InnoDB 数据表上的索引是表空间的一个组成部分），它们包含着对数据表里所有记录的**引用指针**。更通俗地说，数据库索引好比是**一本书前面的目录**，能加快数据库的查询速度。
>
> **索引的目的**在于**提高查询效率**。类比字典：要查“mysql”这个单词，我们会先定位到字母 m，再往下找 y，再找剩下的 sql。如果没有索引，就得把所有单词看一遍；如果想找 m 开头的单词呢？或者 ze 开头的单词呢？没有索引根本无法完成。

### 22.3 索引的原理

> 除了词典，生活中随处可见索引的例子，如火车站的车次表、图书的目录等。它们的原理都是一样的：**通过不断缩小想要获得数据的范围来筛选出最终想要的结果，同时把随机的事件变成顺序的事件**，也就是总是通过同一种查找方式来锁定数据。
>
> 数据库要复杂得多，因为不仅面临**等值查询**，还有**范围查询**（`>`、`<`、`between`、`in`）、**模糊查询**（`like`）、**并集查询**（`or`）等。
>
> 思路：能不能把数据**分成段**，然后分段查询？例如 1000 条数据，1~100 为第一段，101~200 为第二段，201~300 为第三段……查第 250 条数据只要找第三段，一下子去除了 90% 的无效数据。

**几种常见索引结构的对比：**

| 结构 | 特点 | 不足 |
| --- | --- | --- |
| 红黑树（二叉查找树） | 查找快 | 树太高；不同节点存在磁盘的**不同位置**，访问 7 个节点就可能需要 2 次磁盘 IO |
| 哈希索引 | 等值查询极快 | 不支持范围查询；需要**连续的磁盘空间**存哈希表，找到较大的连续空间难度大（磁盘碎片严重） |
| **B+ 树索引** | **MySQL（InnoDB）默认**；树矮、磁盘 IO 少；叶子节点**有序并相连**，支持范围查询 | 维护索引有成本 |

> **B+ 树索引的结构（以 `name` 列为例）**：
>
> - 非叶子节点**只存索引键值和指向子节点的指针**，不存实际数据，所以一个节点能放更多键，树更矮
> - **叶子节点**才存放实际数据（或数据的指针），并且叶子节点之间用指针**串成有序链表**，方便范围查询
> - 讲义中的示意：每个索引项约 12 字节（4 字节 `name` 信息 + 4 字节原有数据的指针 `p`，对 B+ 树来说**只有叶子节点才存这个实际数据指针** + 4 字节叶子节点指针）

### 22.4 索引的使用

```sql
-- 查看某张表上的所有索引
show index from 表名;
-- 创建索引：create index 索引名称 on 表名(字段名称(长度))
-- 如果指定字段是字符串，需要指定长度，建议长度与定义字段时的长度一致
-- 如果字段类型不是字符串，可以不填写长度部分
create index 索引名称 on 表名(字段名称(长度));
-- 删除索引
drop index 索引名称 on 表名;
```

> **补充**：
>
> - **主键**（`primary key`）和**唯一约束**（`unique`）会自动创建索引
> - **唯一索引**：`create unique index 索引名 on 表名(列);`，要求列值不重复
> - **联合（复合）索引**：`create index 索引名 on 表名(列1, 列2);`，遵循**最左前缀原则**——查询条件要从最左边的列开始连续使用才能用上索引
> - 用 `explain select ...;` 可以查看一条查询是否使用了索引（看结果中的 `key` 列）

### 22.5 索引演示：十万条数据的查询对比

**① 创建测试表**

```sql
-- 创建只有一个字段的测试表
create table test_index(title varchar(10));
```

**② 用 Python 向表中加入十万条数据**

```python
from pymysql import connect  # 导入 connect 函数，用于创建数据库连接


def main():
    """向 test_index 表中插入 10 万条数据，用于对比索引前后的查询耗时。"""
    # 创建 Connection 连接对象，参数含义见 1.5
    conn = connect(host='localhost', port=3306, database='jing_dong',
                   user='root', password='mysql', charset='utf8')
    # 获得 Cursor 对象
    cursor = conn.cursor()
    # 插入 10 万次数据，i 从 0 到 99999
    for i in range(100000):
        # 每次插入 'ha-0'、'ha-1'、...、'ha-99999'
        cursor.execute("insert into test_index values('ha-%d')" % i)
    # 提交数据：不提交的话插入不会生效
    conn.commit()
    cursor.close()  # 关闭游标
    conn.close()  # 关闭连接


if __name__ == '__main__':
    main()
```

**③ 对比查询时间**

```sql
-- 开启运行时间监测（会话级别，记录每条语句的耗时；该功能在 8.0 中已标记为过时，但仍可用于学习）
set profiling=1;
-- 查找最后一条数据 ha-99999（没有索引：需要全表扫描 10 万行）
select * from test_index where title='ha-99999';
-- 查看语句执行的时间
show profiles;
-- 为表 test_index 的 title 列创建索引，长度 10 与字段定义一致
create index title_index on test_index(title(10));
-- 再次执行同样的查询（有索引：B+ 树快速定位）
select * from test_index where title='ha-99999';
-- 再次查看执行时间，会发现有索引时耗时大幅下降
show profiles;
```

### 22.6 索引类型

> - **哈希索引**：通过哈希函数定位，适合等值查询，不支持范围查询（`Memory` 引擎使用；InnoDB 有自适应哈希索引，由系统自动管理）
> - **B 树 / B+ 树索引**：默认创建的索引就是 **B+ 树索引**
> - **聚集索引与非聚集索引**：
>   - **聚集索引**：索引和数据**存放在一起**，叶子节点就是数据行本身；InnoDB 的**主键索引**就是聚集索引，所以数据按主键顺序物理存储（对应约束表里“主键：物理上存储的顺序”）
>   - **非聚集（二级）索引**：叶子节点存的是**主键值**，查询时先查到主键，再回到聚集索引查完整数据（**回表**）
>
> 详细资料：[聚集索引和非聚集索引](https://blog.csdn.net/hollis_chuang/article/details/102908445)。

### 22.7 使用索引的注意事项

> - 建立**太多的索引**将会影响**更新和插入**的速度，因为数据变化时需要**同步更新每个索引文件**。对于一个**经常需要更新和插入**的表格，就没有必要为一个**很少使用**的 `where` 子句单独建立索引
> - 对于**比较小的表**，排序的开销不会很大，也没有必要建立另外的索引
> - 建立索引会**占用磁盘空间**
> - 索引的收益与代价：

| 操作 | 影响 |
| --- | --- |
| 查询（带 `where`、`order by`、连接条件） | **大幅提高** |
| 带 `where` 条件的修改、删除 | 提高（定位要操作的行更快） |
| 新增 | 降低一点（需要同时维护索引） |
| 修改被索引的列 | 降低（需要同时维护索引） |
| 磁盘空间 | 额外占用 |

> **哪些列适合建索引**：经常出现在 `where`、`order by`、`group by`、`join on` 中的列；区分度高（重复值少）的列。
> **哪些列不适合**：重复值很多的列（如性别）、很少被查询的列、频繁更新的列。

---

## 二十三、账户管理与权限

> 在**生产环境**下操作数据库时，**绝对不可以使用 root 账户连接**，而是创建**特定的账户**，授予这个账户**特定的操作权限**，然后连接进行操作，主要的操作就是数据的 CRUD。
>
> 这一章属于**了解**内容（除非做 DBA）。

### 23.1 MySQL 的账户体系

> 根据账户所具有的权限不同，MySQL 的账户可以分为以下几种：
>
> ---

| 级别 | 说明 |
| --- | --- |
| 服务实例级账号 | 启动了一个 `mysqld`，即为一个数据库实例；如果某用户（如 `root`）拥有服务实例级分配的权限，那么该账号就可以删除所有的数据库，连同这些库中的表 |
| 数据库级别账号 | 对特定数据库执行增删改查的所有操作 |
| 数据表级别账号 | 对特定表执行增删改查等所有操作 |
| 字段级别的权限 | 对某些表的特定字段进行操作 |
| 存储程序级别的账号 | 对存储程序（存储过程、函数等）进行增删改查的操作 |

> ---
>
> 账户的操作主要包括：**创建账户、删除账户、修改密码、授权权限**等。
>
> **注意**：
>
> 1. 进行账户操作时，需要使用 **root 账户**登录，这个账户拥有最高的实例级权限
> 2. 通常都使用**数据库级**的操作权限

### 23.2 查看所有用户

> 所有用户及权限信息存储在 `mysql` 数据库的 `user` 表中。主要字段说明：
>
> - `Host`：表示允许访问的主机
> - `User`：用户名
> - `authentication_string`：密码，**加密后的值**

```sql
-- 切换到系统库 mysql
use mysql;
-- 查看 user 表的结构
desc user;
-- 查看所有用户：host（允许从哪里登录）、user（用户名）、authentication_string（加密后的密码）
select host,user,authentication_string from user;
```

```
mysql> select host,user,authentication_string from user;
+-----------+------------------+-------------------------------------------+
| host      | user             | authentication_string                     |
+-----------+------------------+-------------------------------------------+
| localhost | root             | *E74858DB86EBA20BC33D0AECAE8A8108C56B17FA |
| localhost | mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| localhost | debian-sys-maint | *EFED9C764966EDB33BB7318E1CBD122C0DFE4827 |
+-----------+------------------+-------------------------------------------+
3 rows in set (0.00 sec)
```

### 23.3 创建账户并授权

> 需要使用**实例级账户**登录后操作，以 `root` 为例。
>
> **常用权限**：`create`、`alter`、`drop`、`insert`、`update`、`delete`、`select`；如果要分配**所有权限**，可以使用 `all privileges`。

**授权语法（5.7 及以前，可一步完成建账户和授权）：**

```sql
-- grant 权限列表 on 数据库.表 to '用户名'@'访问主机' identified by '密码';
grant 权限列表 on 数据库 to '用户名'@'访问主机' identified by '密码';
```

> **访问主机**的取值：
>
> - `%`：此账户可以使用**任何 IP** 的主机登录访问
> - `localhost`：只允许**本机**访问
> - 具体 IP（如 `192.168.1.10`）：只允许该主机访问
>
> **数据库.表** 的写法：`jing_dong.*` 表示 `jing_dong` 库的所有表；`*.*` 表示所有库的所有表。

**示例 1**：创建一个 `laowang` 的账号，密码为 `123456`，**只能通过本地访问**，并且**只能对 `jing_dong` 数据库中的所有表进行读操作**。

```bash
# step1：使用 root 登录，回车后输入密码，再回车
$mysql -uroot -p
```

```sql
-- step2：创建账户并授予权限（8.0 必须分两步：先 create user，再 grant）
-- 创建用户 laowang，只能从本机登录，密码 123
create user 'laowang'@'localhost' identified by '123';
-- 授予 laowang 对 jing_dong 库所有表的 select（查询）权限
grant select on jing_dong.* to 'laowang'@'localhost';
-- 查看 laowang 拥有哪些权限
show grants for laowang@localhost;
-- step3：退出 root 的登录
quit
```

```bash
# step4：使用 laowang 账户登录，回车后输入密码；登录后只能查询 jing_dong 库
$mysql -ulaowang -p
```

**示例 2**：创建一个 `laoli` 的账号，密码为 `12345678`，**可以在任意电脑上连接访问**，并且对 `jing_dong` 数据库中的所有表拥有**所有权限**。

```sql
-- 5.7 版本：一条语句同时创建账户并授权
grant all privileges on jing_dong.* to 'laoli'@'%' identified by '12345678';

-- 8.0 版本：必须分两步
create user 'laoli'@'%' identified by '12345678';
grant all privileges on jing_dong.* to 'laoli'@'%';
```

> **MySQL 8.0 与 5.7 的差异**：8.0 中 `grant ... identified by '密码'` 的写法已经**被移除**（会报语法错误），必须先 `create user` 创建账户，再 `grant` 授权。

### 23.4 修改权限、回收权限

```sql
-- 修改（追加）权限：在原有基础上给 账户@主机 追加权限；with grant option 表示该账户还可以把权限授予别人
grant 权限名称 on 数据库 to 账户@主机 with grant option;
-- 举例：给 laowang 追加 jing_dong 库的 insert、update 权限
grant insert,update on jing_dong.* to 'laowang'@'localhost';
-- 回收权限（补充）：revoke 权限 on 数据库 from 账户@主机
revoke insert,update on jing_dong.* from 'laowang'@'localhost';
```

> `grant` / `revoke` 执行后权限通常即时生效；如果是直接修改 `mysql.user` 等系统表，则必须执行 `flush privileges;` 刷新。

### 23.5 修改密码

> 使用 `root` 登录后修改。以前（5.7）的做法是修改 `mysql.user` 表并使用 `password()` 函数加密；**推荐使用 `alter user` 语句**（5.7.6+ 和 8.0 均支持）。

```sql
-- 修改用户的密码：alter user '用户名'@'主机' identified by '新密码';
alter user 'laowang'@'localhost' identified by 'luke123';
-- 刷新权限，使修改立即生效
flush privileges;
```

### 23.6 远程登录（危险，慎用）

> 远程登录的配置见 [3.2 开启远程连接](#32-开启远程连接)。下面补充两种方法的原理和排查。

**方法 1：配置修改法（仅了解，不要在生产环境使用）**

> 如果要在一台 Ubuntu 中使用 `mysql` 命令远程连接另外一台 MySQL 服务器：

```bash
# 修改 MySQL 服务端的配置文件，把 bind-address 改为 0.0.0.0（或注释掉）
$vim /etc/mysql/mysql.conf.d/mysqld.cnf
# 重启 MySQL 服务
$service mysql restart
# 在另外一台机器上用 -h 指定服务器 IP 进行连接测试
$mysql -h 服务器IP -uroot -p
```

**如果依然连不上，可能的原因：**

1. **网络不通**：用 `ping xxx.xxx.xxx.xxx` 检查网络是否正常
2. **`bind_address` 参数**：本地登录数据库，查看 `my.cnf` 文件，以及当前参数 `show variables like 'bind_address';`；如果是 `127.0.0.1`，那么只能本地登录
3. **`skip_networking` 参数**：如果设置了该参数，那么只能本地登录 MySQL 数据库
4. **端口是否正确**：默认 3306，确认没有被修改，且防火墙放行

**方法 2：改表法**

> 如果账号不允许从远程登录，只能在 `localhost`，这时只要在 `localhost` 那台电脑登录 MySQL 后，更改 `mysql` 数据库里 `user` 表的 `host` 项，从 `localhost` 改成 `%`。

```bash
# 登录 MySQL
$mysql -u root -p
```

```sql
-- 切换到系统库
use mysql;
-- 把 root 允许登录的主机改为 %（任意主机）
update user set host = '%' where user = 'root';
-- 查看修改结果
select host, user from user;
-- 操作结束之后必须刷新权限
flush privileges;
```

### 23.7 删除账户

```sql
-- 语法 1（推荐）：使用 root 登录，drop user '用户名'@'主机';
drop user 'laowang'@'%';
-- 语法 2：直接删除 mysql 数据库的 user 表中的数据（语法 1 失败时才采用）
delete from user where user='laowang';
-- 操作结束之后必须刷新权限
flush privileges;
```

> **推荐使用语法 1** 删除用户，如果语法 1 删除失败，再采用语法 2。

### 23.8 忘记 root 密码怎么办

> 一般也轮不到我们来管理 `root` 账户，这里只给出思路（万一遇到，参考 [忘记 root 密码的处理方法](http://blog.csdn.net/lxpbs8851/article/details/10895085)）：
>
> 1. 停止 MySQL 服务
> 2. 以**跳过权限验证**的方式启动 MySQL（`mysqld_safe --skip-grant-tables &`），此时不输入密码即可登录
> 3. 登录后先 `flush privileges;`，再用 `alter user 'root'@'localhost' identified by '新密码';` 重设密码
> 4. 正常重启 MySQL 服务，用新密码登录
>
> **注意**：跳过权限验证期间任何人都可以连接数据库，操作要尽快完成，并确保网络安全。

---

## 二十四、主从同步

### 24.1 主从同步的定义与好处

> **主从同步**使得数据可以从一个数据库服务器**复制**到其他服务器上。在复制数据时，一个服务器充当**主服务器**（master），其余的服务器充当**从服务器**（slave）。
>
> 因为复制是**异步**进行的，所以从服务器不需要一直连接着主服务器，从服务器甚至可以通过拨号断断续续地连接主服务器。通过配置文件，可以指定复制**所有的数据库、某个数据库，甚至是某个数据库上的某个表**。
>
> **使用主从同步的好处：**
>
> - **读写分离，提高性能**：通过增加从服务器来提高数据库的性能，在主服务器上执行**写入和更新**，在从服务器上向外提供**读**功能，可以动态地调整从服务器的数量，从而调整整个数据库的性能
> - **提高数据安全**：数据已复制到从服务器，从服务器可以终止复制进程，所以可以**在从服务器上备份而不破坏主服务器**相应数据
> - **分担主服务器压力**：在主服务器上生成实时数据，而在从服务器上**分析这些数据**，从而提高主服务器的性能

### 24.2 主从同步的机制

> MySQL 服务器之间的主从同步是基于**二进制日志**（binlog）机制：
>
> - **主服务器**使用二进制日志来**记录数据库的变动情况**（所有写操作）
> - **从服务器**通过**读取和执行**该日志文件来保持和主服务器的数据一致
> - 从服务器会接收到该日志的一个副本，可以指定执行日志中的哪一类事件（譬如只插入数据或者只更新数据），**默认会执行日志中的所有语句**
> - 每一个从服务器会记录关于二进制日志的信息：**文件名**和**已经处理过的位置**，这样不同的从服务器可以分别执行同一个二进制日志的不同部分，并且从服务器可以随时连接或者中断和主服务器的连接
> - 主服务器和每一个从服务器都必须配置一个**唯一的 ID 号**（在 `my.cnf` 文件的 `[mysqld]` 模块下的 `server-id` 配置项）
> - 另外，每一个从服务器还需要通过 `CHANGE MASTER TO` 语句来配置它要连接的主服务器的 IP 地址、日志文件名称和该日志里面的位置

**主从同步的数据流（简图）：**

```
主服务器 master                           从服务器 slave
 ┌──────────────┐                        ┌──────────────────────────┐
 │ 写操作       │                        │ IO 线程：拉取主库 binlog  │
 │   ↓          │      网络传输           │   ↓                      │
 │ 二进制日志   │ ─────────────────────→ │ 中继日志 relay log       │
 │ (binlog)     │                        │   ↓                      │
 └──────────────┘                        │ SQL 线程：重放日志中的语句 │
                                         └──────────────────────────┘
```

### 24.3 配置主从同步的基本步骤

> 有很多种配置主从同步的方法，可以总结为如下步骤：
>
> 1. 在**主服务器**上，必须开启**二进制日志**机制，并配置一个**独立的 ID**
> 2. 在**每一个从服务器**上，配置一个**唯一的 ID**；在主服务器上创建一个用来**专门复制主服务器数据的账号**
> 3. 在开始复制进程前，在主服务器上**记录二进制文件的位置信息**（文件名和位置）
> 4. 如果在开始复制之前，数据库中**已经有数据**，就必须先创建一个**数据快照**（可以使用 `mysqldump` 导出数据库，或者直接复制数据文件），还原到从服务器
> 5. 配置从服务器要连接的主服务器的 **IP 地址、登录授权、二进制日志文件名和位置**

### 24.4 详细配置主从同步

> 主和从的身份可以自己指定。这里将**虚拟机 Ubuntu 中的 MySQL 作为主服务器**，将 **Windows 中的 MySQL 作为从服务器**。在主从设置前，要保证 Ubuntu 与 Windows 间的**网络连通**。

#### 24.4.1 备份主服务器原有数据到从服务器

> 如果在设置主从同步前，主服务器上已有大量数据，可以使用 `mysqldump` 进行数据备份并还原到从服务器，以实现数据的复制。

**在主服务器 Ubuntu 上备份：**

```bash
# mysqldump：导出数据库为 SQL 文件
# -u：用户名；-p：后面直接写密码（这里是 mysql，-p 与密码之间不能有空格）
# --all-databases：导出所有数据库
# --lock-all-tables：执行备份时锁住所有表，防止备份过程中有数据被修改，保证数据一致
# > ~/master_db.sql：把导出的内容保存到 ~/master_db.sql 文件（位置可自己指定）
$mysqldump -uroot -pmysql --all-databases --lock-all-tables > ~/master_db.sql
```

**在从服务器 Windows 上还原：**

> 1. 找到 Windows 上 `mysql` 命令所在的位置（在 MySQL 安装目录的 `bin` 目录下），在该目录打开命令窗口
> 2. 将从主服务器 Ubuntu 中导出的文件**复制**到从服务器 Windows 中，可以放在上面 `mysql` 命令所在的文件夹中，方便还原
> 3. 在命令窗口中执行还原操作：

```bash
# 在 Windows 命令窗口中执行：-u 用户名，-p 密码，< 表示从 master_db.sql 文件读入并执行
$mysql -uroot -pmysql < master_db.sql
```

> 还原完成后，查看表是否发生变化。**注意要关闭连接，再次打开连接**进行查看。

#### 24.4.2 配置主服务器 master（Ubuntu 中的 MySQL）

**① 编辑配置文件，设置 `log_bin` 和 `server-id`**

```bash
# 编辑 MySQL 服务端配置文件
$sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

```
[mysqld]
# 服务器的唯一 ID，主服务器设为 1（整数，主从各不相同）
server-id = 1
# 开启二进制日志，并指定日志文件的位置和前缀
log_bin = /var/log/mysql/mysql-bin.log
# 以下为可选项：
# 二进制日志过期天数（8.0 中已过时，改用 binlog_expire_logs_seconds）
expire_logs_days = 10
# 单个二进制日志文件的最大大小
max_binlog_size = 100M
# 只记录哪个数据库的二进制日志（要同步多个数据库时，重复写多行即可）
binlog_do_db = test
# 忽略哪个数据库的二进制日志
binlog_ignore_db = mysql
```

> **注意**：MySQL 8.0 默认已经开启二进制日志，`server-id` 默认也有值，但**仍应显式配置**，保证主从 ID 不同。

**② 重启 MySQL 服务**

```bash
# 修改配置后必须重启才能生效
$sudo service mysql restart
```

**③ 登录主服务器的 MySQL，创建用于同步数据的账号**

```bash
# 登录主服务器的 MySQL
$mysql -uroot -pmysql
```

```sql
-- 创建复制专用账号 slave，密码 123；@'%' 表示允许从任意主机（也就是从服务器）登录
create user 'slave'@'%' identified by '123';
-- 授予“复制从库”权限：复制需要的是 replication slave 权限，*.* 表示所有库所有表
grant replication slave on *.* to 'slave'@'%';
-- 刷新权限
flush privileges;
```

> **说明**：讲义中的写法是 `create user 'slave'@'localhost'`、`grant select on *.* to ...`，再 `update user set host='%'`（使其能远程登录）。但**复制账号真正需要的权限是 `REPLICATION SLAVE`**，仅有 `select` 权限无法完成复制，且直接创建 `@'%'` 账号比修改 `host` 更规范，所以这里采用上面的写法。

**④ 获取主服务器的二进制日志信息**

```sql
-- 查看主服务器当前使用的二进制日志文件和位置（8.0.22+ 也可用 show binary log status;）
show master status;
```

> 结果中：
>
> - `File`：当前使用的日志文件名（如 `mysql-bin.000006`）
> - `Position`：当前使用的文件位置（如 `590`）
>
> 这两个参数**必须记下**，配置从服务器时会用到。

#### 24.4.3 配置从服务器 slave（Windows 中的 MySQL）

**① 找到 Windows 中 MySQL 的配置文件 `my.ini`，将 `server-id` 修改为 2，保存退出**

```
[mysqld]
# 从服务器的唯一 ID，必须与主服务器不同
server-id = 2
```

> 也可以在 MySQL 中临时设置：`set global server_id = 2;`（重启后失效，需要永久生效仍要改配置文件）。

**② 重启 MySQL 服务**

> 在开始菜单中输入 `services.msc`，打开 Windows 服务管理，找到 `MySQL57`（或对应版本的服务名），**重启**该服务。

**③ 进入 Windows 的 MySQL，设置连接到主服务器**

```sql
-- change master to：告诉从服务器要从哪个主服务器、哪个日志文件的哪个位置开始同步
change master to
    -- 主服务器 Ubuntu 的 IP 地址
    master_host = '10.211.55.5',
    -- 主服务器上创建的复制账号
    master_user = 'slave',
    -- 复制账号的密码
    master_password = '123',
    -- 前面 show master status 查到的日志文件名
    master_log_file = 'mysql-bin.000006',
    -- 前面 show master status 查到的日志位置
    master_log_pos = 590;
```

> **8.0.23 以后的新写法**：`CHANGE MASTER TO` 改为 `CHANGE REPLICATION SOURCE TO`，参数名也相应改为 `SOURCE_HOST`、`SOURCE_USER`、`SOURCE_PASSWORD`、`SOURCE_LOG_FILE`、`SOURCE_LOG_POS`；`START SLAVE` 改为 `START REPLICA`；`SHOW SLAVE STATUS` 改为 `SHOW REPLICA STATUS`。旧写法在 8.0 中仍可用，但已过时。

**④ 开启同步，查看同步状态**

```sql
-- 开启主从同步线程（关闭同步用 stop slave;）
start slave;
-- 查看同步状态：\G 让结果竖着显示，更易阅读
show slave status\G
```

> **如何判断同步成功**：在 `show slave status\G` 的结果中，查看这两项**都为 `Yes`**：
>
> - `Slave_IO_Running: Yes`：IO 线程正常（能从主库拉取日志）
> - `Slave_SQL_Running: Yes`：SQL 线程正常（能执行日志中的语句）
>
> 如果有 `No`，看同一结果中的 `Last_IO_Error`、`Last_SQL_Error` 排查原因（常见：网络不通、账号密码错误、日志文件名或位置写错、`server-id` 冲突）。

#### 24.4.4 测试主从同步

```sql
-- 在 Ubuntu 的 MySQL（主服务器）中创建一个数据库
create database test_sync charset=utf8;
```

```sql
-- 在 Windows 的 MySQL（从服务器）中查看，如果能看到 test_sync，说明主从同步成功
show databases;
```

> **后续**：此后在主服务器上的所有写操作（建库、建表、增删改）都会自动同步到从服务器。**注意**：不要直接在从服务器上写数据，否则会造成主从数据不一致（可以把从库设置为只读：`set global read_only=1;`）。
>
> 补充资料：[主从同步配置教程](https://blog.csdn.net/wd2014610/article/details/79715974)；MySQL 8.0 配置账户密码方式不同，参考 [8.0 配置密码](https://blog.csdn.net/qq_23859799/article/details/85862821)。

---

## 二十五、速查

### 25.1 SQL 语句速查

| 类别 | 语句 | 说明 |
| --- | --- | --- |
| 库 | `show databases;` | 查看所有数据库 |
| 库 | `use 库名;` | 使用数据库 |
| 库 | `select database();` | 查看当前使用的数据库 |
| 库 | `create database 库名 charset=utf8;` | 创建数据库 |
| 库 | `show create database 库名;` | 查看库的创建语句 |
| 库 | `drop database 库名;` | 删除数据库 |
| 表 | `show tables;` | 查看当前库的所有表 |
| 表 | `desc 表名;` | 查看表结构 |
| 表 | `show create table 表名;` | 查看表的创建语句 |
| 表 | `create table 表名(列 类型 约束,...);` | 创建表 |
| 表 | `alter table 表名 add 列 类型;` | 添加字段 |
| 表 | `alter table 表名 change 原名 新名 类型;` | 修改字段（可重命名） |
| 表 | `alter table 表名 modify 列 类型;` | 修改字段类型 |
| 表 | `alter table 表名 drop 列;` | 删除字段 |
| 表 | `drop table 表名;` | 删除表 |
| 增 | `insert into 表名 values(...);` | 全列插入（主键用 0 占位） |
| 增 | `insert into 表名(列,...) values(...),(...);` | 部分列、多行插入 |
| 改 | `update 表名 set 列=值 where 条件;` | 修改数据 |
| 删 | `delete from 表名 where 条件;` | 删除数据 |
| 查 | `select 列 from 表名 where 条件;` | 条件查询 |
| 查 | `select distinct 列 from 表名;` | 去重 |
| 查 | `order by 列 asc\|desc` | 排序 |
| 查 | `count() max() min() sum() avg()` | 聚合函数 |
| 查 | `group by 列 having 条件` | 分组与分组后过滤 |
| 查 | `limit start,count` | 分页（start 从 0 开始） |
| 查 | `表1 inner\|left\|right join 表2 on 条件` | 连接查询 |
| 视图 | `create view v_名 as select ...;` / `drop view v_名;` | 创建 / 删除视图 |
| 事务 | `begin;` / `commit;` / `rollback;` | 开启 / 提交 / 回滚事务 |
| 索引 | `show index from 表;` / `create index 名 on 表(列(长度));` / `drop index 名 on 表;` | 查看 / 创建 / 删除索引 |
| 账户 | `create user '用户'@'主机' identified by '密码';` | 创建账户（8.0） |
| 账户 | `grant 权限 on 库.表 to '用户'@'主机';` | 授权 |
| 账户 | `show grants for 用户@主机;` | 查看权限 |
| 账户 | `alter user '用户'@'主机' identified by '新密码';` | 修改密码 |
| 账户 | `drop user '用户'@'主机';` | 删除账户 |
| 账户 | `flush privileges;` | 刷新权限 |

### 25.2 Shell 命令速查

```bash
# 启动 / 停止 / 重启 / 查看 MySQL 服务状态
$sudo service mysql start
$sudo service mysql stop
$sudo service mysql restart
$sudo service mysql status

# 登录 MySQL；-u 用户名，-p 提示输入密码，-h 主机，-P 端口
$mysql -u root -p
$mysql -h 127.0.0.1 -P 3306 -u root -p

# 备份数据库为 SQL 文件
$mysqldump -uroot -p 库名 > 库名.sql

# 把 SQL 文件还原到已存在的数据库中
$mysql -uroot -p 库名 < 库名.sql
```

### 25.3 学习总结

> - **SQL 分类**：`DQL` 查、`DML` 增删改、`DDL` 建库建表、`TPL` 事务、`DCL` 权限
> - **设计表**：遵循三范式，一对多在“多”的一方加外键字段，多对多用中间表；重要数据用**逻辑删除**
> - **类型选择**：够用就行；金额用 `decimal`；固定长度用 `char`，不固定用 `varchar`；文件存路径，不存文件本身
> - **查询执行顺序**：`from → where → group by → having → select → order by → limit`
> - **`where` 过滤行，`having` 过滤组**；聚合函数不能用在 `where` 中
> - **连接查询**：内连接取交集，左连接保留左表全部，右连接保留右表全部
> - **事务**：多个修改要么全成功要么全失败；`begin → ... → commit / rollback`；引擎必须是 InnoDB
> - **索引**：查询利器，但会降低写入速度并占用空间；给查询频繁、区分度高的列建索引
> - **权限**：生产环境不用 root；按需授予最小权限；8.0 需先 `create user` 再 `grant`
> - **主从同步**：基于二进制日志，主写从读，`server-id` 必须唯一，用 `show slave status\G` 检查两个线程是否都是 `Yes`
