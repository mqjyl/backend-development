# MySQL

MySQL 由连接池、SQL 接口、解析器、优化器、缓存、存储引擎等组成，可以分为三层，即 MySQL Server 层、存储引擎层和文件系统层。MySQL Server 层又包括连接层（网络接入层）和 SQL 层。如下是官方文档中 MySQL 的基础架构图：

![](<../../.gitbook/assets/image (6).png>)

### :pen\_fountain: 1、网络接入层

主要负责连接管理、授权认证、安全等等。每个客户端连接都对应着服务器上的一个线程。服务器上维护了一个线程池，避免为每个连接都创建销毁一个线程。当客户端连接到MySQL服务器时，服务器对其进行认证。可以通过用户名与密码认证，也可以通过`SSL`证书进行认证。登录认证后，服务器还会验证客户端是否有执行某个查询的操作权限。这一层并不是MySQL所特有的技术。

应用程序通过接口（如 `ODBC`、`JDBC`）来连接 MySQL，最先连接处理的是连接层。连接层包括通信协议、线程处理、用户名密码认证 3 部分。

* 通信协议负责检测客户端版本是否兼容 MySQL 服务端。
* 线程处理是指每一个连接请求都会分配一个对应的线程，相当于一条 SQL 对应一个线程，一个线程对应一个逻辑 CPU，在多个逻辑 CPU 之间进行切换。
* 密码认证用来验证用户创建的账号、密码，以及 host 主机授权是否可以连接到 MySQL 服务器。

\
Connection Pool（连接池）属于连接层。由于每次建立连接都需要消耗很多时间，连接池的作用就是将用户连接、用户名、密码、权限校验、线程处理等需要缓存的需求缓存下来，下次可以直接用已经建立好的连接，提升服务器性能。

> 为什么要设计成线程池？
>
> 在服务器内部，每个client都要有自己的线程。这个连接的查询都在一个单独的线程中执行。想象现实场景中数据库访问连接实在是太多了，如果每次连接都要创建一个线程，同时还要负责该线程的销毁。对于系统来说是多么大的消耗。由于线程是操作系统宝贵的资源。这时候线程池的出现就显得自然了，服务器缓存了线程，因此不需要为每个Client连接创建和销毁线程。&#x20;

### :pen\_fountain: 2、SQL层

SQL层是MySQL的核心，MySQL的核心服务层都在这一层，权限判断，查询解析，SQL执行计划分析，SQL执行计划优化，查询缓存。以及跨存储引擎的功能都在这一层实现：存储过程，触发器，视图等。

* 权限判断可以审核用户有没有访问某个库、某个表，或者表里某行数据的权限。
* 查询缓存通过 Query Cache 进行操作，如果数据在 Query Cache 中，则直接返回结果给客户端，不必再进行查询解析、优化和执行等过程。
* 查询解析器针对 SQL 语句进行解析，判断语法是否正确。
* 预处理器对解析器无法解析的语义进行处理。
* 查询优化器对 SQL 进行改写和相应的优化，并生成最优的执行计划，就可以调用程序的 API 接口，通过存储引擎层访问数据。

> 执行计划：执行计划（execution plan，也叫查询计划或者解释计划）是数据库执行 SQL 语句的具体步骤，例如通过索引还是全表扫描访问表中的数据，连接查询的实现方式和连接的顺序等。如果 SQL 语句性能不够理想，我们首先应该查看它的执行计划。

Management Services & Utilities、SQL Interface、Parser、Optimizer 和 Caches & Buffers 属于 SQL 层，详细说明如下表所示：

| 名称                               | 说明                                                                                                                                                                                                                                                                                         |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Management Services & Utilities  | MySQL 的系统管理和控制工具，包括备份恢复、MySQL 复制、集群等。                                                                                                                                                                                                                                                      |
| SQL Interface（SQL 接口）            | 用来接收用户的 SQL 命令，返回用户需要查询的结果。例如 SELECT FROM 就是调用 SQL Interface。                                                                                                                                                                                                                              |
| Parser（查询解析器）                    | 在 SQL 命令传递到解析器的时候会被解析器验证和解析，以便 MySQL 优化器可以识别的数据结构或返回 SQL 语句的错误。                                                                                                                                                                                                                            |
| Optimizer（查询优化器）                 | <p>SQL 语句在查询之前会使用查询优化器对查询进行优化，同时验证用户是否有权限进行查询，缓存中是否有可用的最新数据。它使用“选取-投影-连接”策略进行查询。<br><br>例如 <code>SELECT id, name FROM student WHERE gender = "女";</code>语句中，SELECT 查询先根据 WHERE 语句进行选取，而不是将表全部查询出来以后再进行 gender 过滤。SELECT 查询先根据 id 和 name 进行属性投影，而不是将属性全部取出以后再进行过滤，将这两个查询条件连接起来生成最终查询结果。</p> |
| Caches & Buffers（查询缓存）           | 如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的，比如表缓存、记录缓存、key 缓存、权限缓存等。                                                                                                                                                                                                               |

通过下图来观察服务层的内部结构：

![](<../../.gitbook/assets/image (3).png>)

> SQL语句在服务层中具体的流程：
>
> 查询缓存：在解析查询之前，服务器会检查查询缓存，如果能找到对应的查询，服务器不必进行查询解析、优化和执行的过程，直接返回缓存中的结果集。
>
> 解析器与预处理器：MySQL会解析查询，并创建了一个内部数据结构（解析树）。这个过程解析器主要通过语法规则来验证和解析。比如SQL中是否使用了错误的关键字或者关键字的顺序是否正确等等。预处理会根据MySQL的规则进一步检查解析树是否合法。比如要查询的数据表和数据列是否存在等。
>
> 查询优化器：优化器将其转化成查询计划。多数情况下，一条查询可以有很多种执行方式，最后都返回相应的结果。优化器的作用就是找到这其中最好的执行计划。优化器并不关心使用的什么存储引擎，但是存储引擎对优化查询是有影响的。优化器要求存储引擎提供容量或某个具体操作的开销信息来评估执行时间。
>
> 查询引擎：在完成解析和优化阶段以后，MySQL会生成对应的执行计划，查询执行引擎根据执行计划给出的指令调用存储引擎的接口得出结果。

### :pen\_fountain: 3、存储引擎层

负责MySQL中数据的存储与提取。 服务器中的查询执行引擎通过API与存储引擎进行通信，通过接口屏蔽了不同存储引擎之间的差异。MySQL采用插件式的存储引擎。MySQL为我们提供了许多存储引擎，每种存储引擎有不同的特点。我们可以根据不同的业务特点，选择最适合的存储引擎。如果对于存储引擎的性能不满意，可以通过修改源码来得到自己想要达到的性能。例如阿里巴巴的**X-Engine，为了满足企业的需求，`facebook`与google都对`InnoDB`存储引擎进行了扩充。**因为在关系数据库中，数据的存储是以表的形式存储的，所以存储引擎也可以称为表类型（即存储和操作此表的类型）。

特点**：**存储引擎是针对于表的而不是针对库的（一个库中不同表可以使用不同的存储引擎），服务器通过API与存储引擎进行通信，用来屏蔽不同存储引擎之间的差异。

### :pen\_fountain: 4、系统文件层

该层主要是将数据库的数据存储在文件系统之上，并完成与存储引擎的交互。
