QPS = req/sec = 请求数/秒

【QPS计算PV和机器的方式】

QPS统计方式 [一般使用 http_load 进行统计]
QPS = 总请求数 / ( 进程总数 *   请求时间 )
QPS: 单个进程每秒请求服务器的成功次数

单台服务器每天PV计算 

 - 公式1：每天总PV = QPS * 3600 * 6
 - 公式2：每天总PV = QPS * 3600 * 8

服务器计算
服务器数量 =   ceil( 每天总PV / 单台服务器每天总PV )

【峰值QPS和机器计算公式】

原理：每天80%的访问集中在20%的时间里，这20%时间叫做峰值时间
公式：( 总PV数 * 80% ) / ( 每天秒数 * 20% ) = 峰值时间每秒请求数(QPS)
机器：峰值时间每秒QPS / 单台机器的QPS   = 需要的机器

问：每天300w PV 的在单台机器上，这台机器需要多少QPS？
答：( 3000000 * 0.8 ) / (86400 * 0.2 ) = 139 (QPS)

问：如果一台机器的QPS是58，需要几台机器来支持？
答：139 / 58 = 3


１.什么是pv　　PV(page view)，即页面浏览量，或点击量；通常是衡量一个网络新闻频道或网站甚至一条网络新闻的主要指标。

　　高手对pv的解释是，一个访问者在24小时(0点到24点)内到底看了你网站几个页面。这里需要强调：同一个人浏览你网站同一个页面，不重复计算pv量，点100次也算1次。说白了，pv就是一个访问者打开了你的几个页面。

　　PV之于网站，就像收视率之于电视，从某种程度上已成为投资者衡量商业网站表现的最重要尺度。

　　pv的计算：当一个访问着访问的时候，记录他所访问的页面和对应的IP，然后确定这个IP今天访问了这个页面没有。如果你的网站到了23点，单纯IP有60万条的话，每个访问者平均访问了3个页面，那么pv表的记录就要有180万条。

有一个可以随时查看PV流量以及你的网站世界排名的工具alexa工具条，安装吧！网编们一定要安装这个。

　　２.什么是uv
　　uv(unique visitor)，指访问某个站点或点击某条新闻的不同IP地址的人数。

　　在同一天内，uv只记录第一次进入网站的具有独立IP的访问者，在同一天内再次访问该网站则不计数。独立IP访问者提供了一定时间内不同观众数量的统计指标，而没有反应出网站的全面活动。

　　３.什么是ＰＲ值
　　PR值，即PageRank，网页的级别技术。取自Google的创始人Larry Page，它是***运算法则（排名公式）的一部分，用来标识网页的等级/重要性。级别从1到10级，10级为满分。PR值越高说明该网页越受欢迎（越重要）。

　　例如：一个PR值为1的网站表明这个网站不太具有流行度，而PR值为7到10则表明这个网站非常受欢迎（或者说极其重要）。

　 　我们可以这样说：一个网站的外部链接数越多其PR值就越高；外部链接站点的级别越高（假如Macromedia的网站链到你的网站上），网站的PR值就 越高。例如：如果ABC.COM网站上有一个XYZ.COM网站的链接，那为ABC.COM网站必须提供一些较好的网站内容，从而Google会把来自 XYZ.COM的链接作为它对ABC.COM网站投的一票。

　　你可以下载和安装Google工具条来检查你的网站级别（PR值）。