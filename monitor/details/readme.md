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


```
Node js 
depandences:
redis
```


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



# 10.207.0.225 (dev)
>	Redis
/usr/local/redis-2.4.17/bin/redis-server
	:6379 /usr/local/redis-2.4.17/etc/m-redis.conf
	:6380 /usr/local/redis-2.4.17/etc/m6380-redis.conf
>	Gearman
/usr/local/gearman/sbin/gearmand
	:4730 
>	?
/usr/local/sinasrv2/sbin/gmond

# 10.207.0.226 (dev)
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



__自动执行定时任务__




创建一个PaaS平台，即维护一套 PaaS 服务构建树

举例 : 内网 dev 环境

* paas_id
	* (角色[])
	* 负载均衡 / 反向代理 [Nginx / Varnish]
	* Web Server [Nginx]
	* Web Server [Apache]
	* Couchbase / Memcached
	* Cache / [Redis]
	* Data / COMDB
	* Data / Mysql
	* VFS / PhotoLib
	* VFS / FS
	* RPC / Gearman
	* SCM / Deploy
	* MM / SSH
	* MM / Monitor


## run flow

client
load base config
	get the server ip:port
	check server connection
setup monitor
	get current server info, and ip
	get services on the server with {ip}
	set server alive && send signal to channel <update:info:{ip}:alive>
	run listen() for listen manage signel
	run monitoring() for automaticly update status
		update the server alive status && send signal to channel
			所有服务器存活信息 <servers:alive> | hash [{ip}:string 1 alive 0 died]
			当前服务器存活信息 <server:alive:{ip}> | string 1 alive 0 died
			服务器状态更新消息 <update:info, json> | channel | publish
				json: { type:'alive', ip:{ip}, alive:{0/1} }
		update the server workload status && send signal to channel
			当前服务器状态更新历史记录 <server:workload:{ip}> | set json
				json: [{ timestamp:'time', info:{...} }, ...]
			服务器状态更新消息 <update:info, "json">
				json: { type:'workload', ip:{ip}, info:{...} }
		update each service status && send signal to channel
			服务存活信息 <service:{ip}:{role}:{instance}:alive> | string 1 alive 0 died
			当前服务器的角色定义 <update:info, json> | channel | publish
				json : { type:'service', role:{role}, instance:{instance}, alive:{0/1}}
			

## Redis Key

	Name space plan

* 所有服务器的基本信息

key : server:info:{ip}
type : hash

* 所有服务器的配置信息

key : server:config:{ip}
type : hash

services : {
	ssh:{
		'_': {
			protected: true
		},
		'default': {
			alive:{
				'cmd': 'ps aux | grep sshd | grep /usr/local/sshd | wc -l'
			}
		}
	},
	mysql:{
		'3301': {
			alive: {
				'cmd': 'ps aux | grep mysqld_safe | grep :port: | wc -l',
			},
			workload: {
				'cmd': 'mysqladmin status --socket=/data0/DB/:port:/mysql.sock',
				'type': 'info'
			}
		},
		'3302': {
			alive: {
				'cmd': 'ps aux | grep mysqld_safe | grep :port: | wc -l',
			},
			workload: {
				'cmd': 'mysqladmin status --socket=/data0/DB/:port:/mysql.sock',
				'type': 'info'
			}
		},
		'3305': {
			alive: {
				'cmd': 'ps aux | grep mysqld_safe | grep :port: | wc -l',
			},
			workload: {
				'cmd': 'mysqladmin status --socket=/data0/DB/:port:/mysql.sock',
				'type': 'info'
			}
		},
		'3306': {
			alive: {
				'cmd': 'ps aux | grep mysqld_safe | grep :port: | wc -l',
			},
			workload: {
				'cmd': 'mysqladmin status --socket=/data0/DB/:port:/mysql.sock',
				'type': 'info'
			}
		},
		'3307': {
			alive: {
				'cmd': 'ps aux | grep mysqld_safe | grep :port: | wc -l',
			},
			workload: {
				'cmd': 'mysqladmin status --socket=/data0/DB/:port:/mysql.sock',
				'type': 'info'
			}
		}
	}
}
controlled: {
	'system': {
		'default': {
			'restart': 'reboot',
			'shutdown': 'shutdown -h now'
		},
	},
	mysql: {
		'default': {
			'start': '/usr/local/sbin/mysql_service start :port:',
			'restart': '/usr/local/sbin/mysql_service restart :port:',
			'stop': '/usr/local/sbin/mysql_service stop :port:'
		}
	}
}

所有服务


* 所有服务的基本信息




