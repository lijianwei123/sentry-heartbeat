###要解决的问题：

1、java应用发布时，需要一段时间内才能够提供正常使用，slb在心跳检测周期会认为应用还是健康,这样会出现短时间的502。

2、java应用发布过程中，现阶段仅是检测进程是否存在，一些应用启动过程出错，因为还存在非daemon线程，java进程是不会挂掉的，这样不能准确告诉发布是否真正成功

解决方案：

参照基于SLB的滚动发布脚本示例

开发了心跳jar

使用方式：


1. 引用jar

2. 配置
spring mvc 需要在spring mvc容器的配置文件加上
<bean class="com.ggj.platform.sentry.heartbeat.HealthUrlConfig"/>
spring boot 不需要配置, 会自动加载配置
 
3. 使用
会自动配置三个接口, 仅限本地调用
 
http://127.0.0.1:port/health/check  心跳检测，配置在slb, 或者jenkis发布成功之后检测应用功能完整(容器完全加载后才会认为功能完整)
http://127.0.0.1:port/health/online 上线，心跳恢复健康
http://127.0.0.1:port/health/offline 下线 心跳异常
 
同时应用启动之后 新建 /data/proc/应用名(app.name)/status 文件，同时文件修改时间更新为当前时间
类似/data/proc/sentry-proxy/status
3、调整发布流程

发布前台应用流程:

 

先调用下线接口->sleep slb能够检测异常间隔时间->关闭应用、删除status文件->发布应用

->调用心跳检测接口 验证应用功能完整->sleep slb能够检测应用正常间隔时间->发布成功

 

发布定时器或者服务中心没有http接口的流程:

关闭应用-> 发布应用-> 检测status文件是否存在->发布成功

