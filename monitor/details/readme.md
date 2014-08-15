# 详细设计


## 模块规划

主要采用 Client / Server 模型

将 `Client端` 安装在需要系统信息采集、系统管控的系统中。
将 `Server端` 安装在应用服务器中。
Server端 使用形式为 Browser / Server 模型，管理者、系统信息查询访客、开发人员，将通过浏览器查看服务器和系统服务的运行状态信息。

         -- fetch sys info -- <Redis> cache ---
[Client] -- listen channel -- <Redis> pub/sub -- [Server]
         -- anlisyze log files -- <MySql> logs --
	
## 关于消息传递

1. 系统状态信息

Client 通过抓取本地服务并匹配服务端的设置，进行服务监控
主要监控以下数据
服务器
Server Info / 周期较长
Network
	ping
	ip change


任务执行的类型
1. 被动监听 : 定时状态更新
2. 主动监听 : 事件触发式的更新
3. 主动执行 : 执行一些简单的管理命令
	1. 内置任务 (服务重启等)
	2. 手工定义 (x 因为安全考虑，暂时不允许使用)

client -> get_client_sysinfo -> connect redis -> <match sysinfo>
-> :not found -> disconnect -> (exit);
-> :found -> update sysinfo -> listen -> dispatch invoke



服务状态检测
### OpenSSH
ps aux | grep /usr/sbin/sshd | wc -l == 0 => [error]

### MySQL 服务是否已经启动
10.207.26.25{0,1}
/usr/local/mysql/bin/mysqladmin status --socket=/data0/DB/{port}/mysql.sock | grep "error" | wc -l == 1 => error
ps aux | grep mysqld | grep {port} | wc -l == 0 => [error]


### Redis 服务检测
redis-cli -h {host} -p {port} info | grep process_id | wc -l == 0 => [error]
process_id:205



# 10.207.0.225
>	Redis
/usr/local/redis-2.4.17/bin/redis-server
	:6379 /usr/local/redis-2.4.17/etc/m-redis.conf
	:6380 /usr/local/redis-2.4.17/etc/m6380-redis.conf
>	Gearman
/usr/local/gearman/sbin/gearmand
	:4730 
>	?
/usr/local/sinasrv2/sbin/gmond

# 10.207.0.226
>	redis
/usr/local/redis-2.4.17/bin/redis-server
	:6379 /usr/local/redis-2.4.17/etc/m-redis.conf
	:6380 /usr/local/redis-2.4.17/etc/m6380-redis.conf
>	Gearman
/usr/local/gearman/sbin/gearmand
	:4730 
	:4731 
	:4732 
	:4733




