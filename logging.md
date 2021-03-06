# 系统日志规范

## 前言

日志是一个系统必不可少的功能模块，其好处就不在这里啰嗦了。


## 最佳实践

比如，用户下单失败了，收到用户的反馈时，能否根据日志快速地找到原因。

需要记录日志的地方：

* 调试信息
* 异常、堆栈跟踪、预期之外的结果（如数据库无法连接）
* 用户进行敏感和重要的数据操作需要写入日志。
* 系统性能瓶颈的位置，记录操作的耗时时间，便于进行系统性能分析。
* 一些耗时操作（执行数据库查询、从磁盘或者存储介质获取数据等）
* 

常见错误相关的日志：

* 系统启动失败
* 系统内部buffer队列满了（在设计正确的系统里面，正常情况下不应该出现buffer满了的情况）

常见需要记录日志的业务：

* 用户登陆、退出（用户IP、用户登录的失败原因和次数）
* 管理员基本操作。
* 一些涉及到金钱的操作（下单，支付等）
* 其他。
* 系统中一些长期执行的任务的执行进度
* 系统启动（记录）

常见性能相关的日志：

* 调用第三方接口（尤其是一些不稳定的第三方）。
* 网络请求
* 链接设备
* 系统初始化、读取配置文件所花时间

常见统计相关的日志

* 用户访问统计
* 用户浏览记录、用户搜索记录（用户数据分析）

### 异常处理日志

如果异常是预期之内的，可以不写或者写 info 日志；
如果这个异常是需要程序员或者运维人员进行处理的否则写 error 日志。


3）Sessions
知道一个问题是由谁引起的非常重要，因此在日志中使用会话标识符就变得必不可少。它可以简单到是一个 IP 地址或者是一个更复杂的 UUID，只要能区分不同的请求者就足够。

### 必要信息要全

一条日志至少包含以下信息：精确到毫秒的时间、日志级别、源文件名、源码行数和日志内容。

##### [强制] 时间必须精确到毫秒级别

##### [建议] 准确适用日志级别

debug：调试时使用，线上关闭。
info：记录一般性的程序运行信息。
warning：
error：用来记录必须处理的错误信息。

### 日志应该是可分析的

好的日志应该既适合阅读又方便工具分析。为了便于 grep 等工具分析，请遵守以下规范：

##### [强制] 日志文件中一条日志只占一行，除非是堆栈打印。

##### [强制] 一条日志中，日志内容放在最后

如：
[com.cjh.eshop.controller.view.TestController:21] - 调试信息。

 
好的日志：
[user1] 用户登录成功。
[user1] 用户成功购买产品 A。
[user2] 订单 003 付款失败。
 
日志
用户1登陆成功
用户1成功购买产品A
用户2订单003付款失败
 
### 合理地拆分日志
不同内容的日志文件应该分开，不要什么日志全都写在一个文件。比如访问日志、调试日志和错误日志分开。
日志不应该按大小来自动划分文件，而是按时间。根据系统的大小，按天数或者小时划分。
Log4j配置按小时划分文件：
log4j.appender.defaultLog=org.apache.log4j.DailyRollingFileAppender
log4j.appender.defaultLog.DatePattern=.yyyy-MM-dd.HH
 
### 异常处理
e.printStackTrace();只会在控制台输出。如果异常是有意义 的，应该写入日志。
 
## 日志管理

* 整个团队（包括开发，运维和测试）定期对记录的日志内容进行Review；
* 必要的备份；
* 开发做运维，通过在查问题的过程来优化日志记录的方式；
* 运维或测试在日志中发现的问题，都需要及时向开发人员反映；

## 日志安全

##### [建议] 注意日志文件及其备份文件的储存位置应该是安全的

##### [建议] 日志中避免出现敏感信息

常见敏感信息如密码、身份证号、余额等。

一些敏感信息可以打马赛克后输出，比如比方字符串部分内容换成*。如：身份证号 422927198806031234,可以考虑这样输出: 42292719880*****34。

## 其他建议

##### [建议] 从问题中完善日志

在线上出现问题的时候，需要尽快发现问题并解决，而同时，需要借此机会好好思考一下当前系统的日志是否合理。需要考虑以下问题：

* 如果定位问题花费了很长时间或者无法定位，那就说明系统日志还存在问题，需要进一步完善和优化
* 需要思考是否可以通过优化日志，来提前预判该问题是否可能发生（如某种资源耗尽而导致的错误，可以对资源的使用情况进行记录）

##### [建议] 在调试中完善日志

开发调试时出现问题，这种情况和线上出现问题是类似的，应该结合日志进行调试。可以参照上一条建议。

## javaWeb 日志

推荐使用SLF4J + log4j 的方式来实现，而不是直接使用log4j，这一点在开发库的时候尤其重要。

SLF4J+logback也不错，据说性能更高，毕竟是log4j创建人后期开发的产品。

* log4j的isDebugEnabled，如果打印信息是常量字符串或简单字符串拼接，那么不需要if ( log.isDebugEnabled())。如果你拼装的动作比较耗资源，请用if ( log.isDebugEnabled() )。最好尽可能用。当然，使用SLF4J可避免这个问题。
* 日志和注释一样，过多过少都不好，需要恰到好处。过多的日志会消耗性能。
* 调试经常使用的System..out.println()，在功能模块完成后，要么删掉，要么日志输出。

* 项目上线后，应该关闭DEBUG级别的日志。
* 不要滥用日志级别。DEGUG的信息是给程序猿看的，INFO的信息是给管理员看的。
* getLogger的参数不要写错，必须是这句代码所在的类。