
## 前言


pgbench是一种在postgres上进行基准测试的简单程序，一般安装后就会自带。pgbench可以再并发的数据库绘画中一遍遍地进行相同序列的SQL语句，并且计算平均事务率。


## 测试准备


既然要测postgres，肯定要先有个postgres。安装过程略过。


一些环境信息：


* postgres版本：15\.3，安装完成后默认配置
* os version：debian 12
* 硬件配置：vbox虚拟机，4核4GB内存，40GB SSD存储


## pgbench选项


运行`pgbench --help`查看帮助文档



```
pgbench is a benchmarking tool for PostgreSQL.

Usage:
  pgbench [OPTION]... [DBNAME]

Initialization options:
  -i, --initialize         调用初始化模式
  -I, --init-steps=[dtgGvpf]+ (default "dtgvp")
                           运行选定的初始化步骤
  -F, --fillfactor=NUM     设置填充因子
  -n, --no-vacuum          初始化期间不运行 VACUUM
  -q, --quiet              安静日志记录（每 5 秒一条消息）
  -s, --scale=NUM          缩放因子
  --foreign-keys           在表之间创建外键约束
  --index-tablespace=TABLESPACE
                           在指定的表空间中创建索引
  --partition-method=(range|hash)
                           使用此方法对 pgbench_accounts 进行分区（默认：range）
  --partitions=NUM         将 pgbench_accounts 分成 NUM 个部分（默认：0）
  --tablespace=TABLESPACE  在指定的表空间中创建表
  --unlogged-tables        将表创建为非记录表

Options to select what to run:
  -b, --builtin=NAME[@W]   添加内置脚本 NAME，权重为 W（默认：1）
                           （使用 "-b list" 列出可用脚本）
  -f, --file=FILENAME[@W]  添加脚本 FILENAME，权重为 W（默认：1）
  -N, --skip-some-updates  跳过 pgbench_tellers 和 pgbench_branches 的更新
                           （与 "-b simple-update" 相同）
  -S, --select-only        仅执行 SELECT 类型事务
                           （与 "-b select-only" 相同）

Benchmarking options:
  -c, --client=NUM         并发数据库客户端数（默认：1）
  -C, --connect            为每个事务建立新连接
  -D, --define=VARNAME=VALUE
                           为自定义脚本定义变量
  -j, --jobs=NUM           线程数（默认：1）
  -l, --log                将事务时间写入日志文件
  -L, --latency-limit=NUM  计数超过 NUM 毫秒的事务为延迟
  -M, --protocol=simple|extended|prepared
                           提交查询的协议（默认：simple）
  -n, --no-vacuum          在测试前不运行 VACUUM
  -P, --progress=NUM       每 NUM 秒显示线程进度报告
  -r, --report-per-command 每个命令报告延迟、失败和重试
  -R, --rate=NUM           目标事务每秒速率
  -s, --scale=NUM          在输出中报告此缩放因子
  -t, --transactions=NUM   每个客户端运行的事务数（默认：10）
  -T, --time=NUM           基准测试持续时间（秒）
  -v, --vacuum-all         测试前清理所有四个标准表
  --aggregate-interval=NUM 将数据汇总在 NUM 秒内
  --failures-detailed      按基本类型分组报告失败情况
  --log-prefix=PREFIX      事务时间日志文件的前缀
                           （默认："pgbench_log"）
  --max-tries=NUM          运行事务的最大尝试次数（默认：1）
  --progress-timestamp     使用 Unix 纪元时间戳作为进度
  --random-seed=SEED       设置随机种子（"time", "rand", 整数）
  --sampling-rate=NUM      日志记录的事务比例（例如，0.01 代表 1%）
  --show-script=NAME       显示内置脚本代码，然后退出
  --verbose-errors         打印所有错误的消息

Common options:
  -d, --debug              打印调试输出
  -h, --host=HOSTNAME      数据库服务器主机或套接字目录
  -p, --port=PORT          数据库服务器端口号
  -U, --username=USERNAME  以指定的数据库用户身份连接
  -V, --version            输出版本信息，然后退出
  -?, --help               显示此帮助信息，然后退出

Report bugs to .
PostgreSQL home page: 

```

## 测试示例


1. 初始化测试数据。以下命令会自动创建四张测试表，在其中一个表`pgbench_accounts`中创建200000行数据。再次初始化的话，pgbench会先把旧表删除，然后再建测试表。



```
pgbench -i -s 2

```

2. 使用默认配置执行一次简单的基准测试



```
$ pgbench

pgbench (15.3)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>  # 本次测试所使用的测试类型
scaling factor: 2  # 用于记录在初始化设置的数据量比例因子
query mode: simple  # 测试是指定的查询类型
number of clients: 1  # 客户端数量
number of threads: 1  # 每个客户端的线程数
maximum number of tries: 1
number of transactions per client: 10  # 每个客户端的事务数
number of transactions actually processed: 10/10  # 实际完成的事务数量和计划完成的事务数量
number of failed transactions: 0 (0.000%)
latency average = 7.501 ms  # 平均响应时间
initial connection time = 1.650 ms
tps = 133.317335 (without initial connection time)

```

删除测试数据



```
DROP TABLE IF EXISTS pgbench_accounts;
DROP TABLE IF EXISTS pgbench_branches;
DROP TABLE IF EXISTS pgbench_history;
DROP TABLE IF EXISTS pgbench_tellers;

```

## 测试数据


500w数据量，postgres为默认配置




| 序号 | 测试命令 | 平均响应时间 | TPS | 总事务数 |
| --- | --- | --- | --- | --- |
| 1 | `pgbench -T60` | 6\.857 ms | 145\.833890 | 8750 |
| 2 | `pgbench -j2 -c4 -T60` | 11\.505 ms | 347\.677286 | 20858 |
| 3 | `pgbench -j4 -c8 -T60` | 13\.721 ms | 583\.052560 | 34987 |


 本博客参考[樱花宇宙官网](https://yzygzn.com)。转载请注明出处！
