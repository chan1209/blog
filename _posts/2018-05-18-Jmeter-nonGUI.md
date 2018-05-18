---
layout: post
title: Jmter 非GUI模式运行

---

### Jmter 非GUI模式运行

1.配置系统路径

追加%JmeterPath%\bin到path环境变量

2.常见命令

例：jmeter -n -t sample.jmx -l log.jtl 

* -h 帮助 -> 打印出有用的信息并退出
* -n 非 GUI 模式 -> 在非 GUI 模式下运行 JMeter
* -t 测试文件 -> 要运行的 JMeter 测试脚本文件
* -l 日志文件 -> 记录结果的文件
* -r 远程执行 -> 启动远程服务
* -H 代理主机 -> 设置 JMeter 使用的代理主机
* -P 代理端口 -> 设置 JMeter 使用的代理主机的端口号

3.建议

用Jmeter做负载测试时，建议写好性能脚本后，用NON GUI模式进行负载测试，即非图形化界面，也就是建议使用命令行运行！因为图形化界面会消耗资源，导致负载测试结果不精确，特别是用图形化界面时还把查看结果树给打开，查看结果树输出的结果很多，所以，**写完负载测试脚本后尽量把查看结果树等调试插件给禁掉**。

![1526613515654](C:\Users\chenpei\AppData\Local\Temp\1526613515654.png) 