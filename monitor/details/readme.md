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
