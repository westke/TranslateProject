用 NTP 把控时间 -- 第一部分:使用概览
============================================================

 ![NTP](https://www.linux.com/sites/lcom/files/styles/rendered_file/public/ntp-time.jpg?itok=zu8dqpki "NTP") 
这系列共三部分，首先，Chirs Binnie考量了一个令人愉快的基础建设中 NTP 服务的重要性。[经许可使用][1]

鲜有网络服务器在计时方面称得上是标准。影响你系统的时间计时的小问题可能花了一两天被发现，这样的不速之客通常会引起连锁效应。

设想你的备份服务器与网络时间协议（NTP）断开连接，过了几天，引起几小时的时间扭曲。你的同事照常九点上班，发现 bandwidth-intensive 类型的备份服务器消耗了所有网络资源，这也就意味着他们在备份完成之前几乎不能登录工作台开始他们的日常工作。

这系列共三部分，首先，我将提供简要介绍 NTP 防止这种困境的发生。从邮件的时间戳到记录你工作的进展，NTP 服务对于一个令人愉快的基础建设是如此重要。

想想非常重要的 NTP 服务器 (其他服务器时间数据来源) 是倒置金字塔的底部，与第1层的服务器（也被称为“主要”服务器）相关。这些服务器（0层，是原子钟和GPS钟之类的装置）与自然时间直接交互。安全沟通的方法很多，例如通过卫星或者无线电。

令人惊讶的是，几乎所有的大型企业都会连接二层服务器（或“次级”服务器）而是不主服务器。如你所料，二层服务器和一层直接同步。如果你觉得大公司可能有自己的本地 NTP 服务器（至少两个，通常三个，为了恢复），这样就会有三层服务器。结果，第3层服务器将连接上层去预定义次级服务器，负责任地传递时间给客户端和服务器作为当前时间的精确反馈。

简单设计构成的 NTP 工作前提是——多亏了英特网通路走过大量地理距离——在信任时间完全准确之前，来回时间（包什么时候发出和多少秒后被收到）都会被清楚记录。设置电脑时间比你想象的要多做很多，如果你不信我，那[这神奇的网站][3]值得一看。

由于有重复访问节点的风险，NTP 如此关键以至于 NTP 与层次服务器之间的连接被期望必须确保内部计时完全被信赖并且能提供额外信息。有一个有用的 Stratum 1 服务器列表在 [主 NTP 站点][4].

正如你在列表所见，一些 NTP 一层服务器以“ClosedAccount”状态运行;这些服务器需要提前同意才可以使用。但是只要你完全按照他们的使用引导做，“OpenAccess” 服务一定会为了轮询开放。每个 “RestrictedAccess” 服务有时候会因为大量客户端访问或者少数轮询间隙而受限。另外有时候会专供某种类型的组织，例如学术界。

### 尊重我的权威

在公共 NTP 服务器上，你可能发现使用引导遵从某些规则。现在让我们看看其中一些。

 “iburst” 选项作用是客户端发送一定数量的包（八个包而不是通常的一个）给 NTP 服务器，轮询间隔会没有应答。
如果在短时间内呼叫 NTP 服务器几次，没有出现可辨识的应答，那么本地时间没有变化。

不像 “iburst” ，按照 NTP 服务器的规则， “burst” 选项一般不允许使用(所以不要用它！)。这个选项不仅在探询间隙发送大量包（明显又是八个），而且也会在能正常使用时这样做。如果你在高层服务器持续发送包，甚至是它们在正常应答时，你可能会因为使用 “burst” 选项而被拉黑。

显然，你连接服务器的频率影响了它加载的速度（和少量带宽使用）。使用“minpoll”和“maxpoll”选项可以本地设置。然而，根据连接 NTP 服务器的规则，你不应该分别修改默认的 64 秒和 1024 秒。

此外，需要提出的是客户应该重视请求时间的服务器发出的 Kiss-Of-Death (KOD) 消息。如果 NTP 服务器不想反馈特殊请求，类似于路由和防火墙技术，那么它极有可能遗弃或吞没每个相关的包。

换句话说,接受异常数据的服务器交互不需要额外的负载而且几乎不消耗流量以至于它认为这不值得回应。你就可以想象,这很无力，有时候礼貌地问客户是否中止或停止比忽略请求更为有效。因此，这种特别的包类型叫做 KOD 包。不受欢迎的 KOD 包被传送给客户端，然后记住这特别的拒绝访问标志。

如果收到不止一个服务器反馈的 KOD 包,客户端会猜想服务器上发生了流量限速的情况(或类似的)。客户端一般会写入本地日志，使用特别服务器差强人意的处理结果，如果你需要分析解决方案。

牢记， NTP 服务器的动态基础建设明显是关键。因此，不要给你的 NTP 配置硬编码 IP 地址。通过使用 DNS 域名，独立服务器衰减网络，服务仍能继续进行，IP 地址空间能被重新分配并且可引入简单的负载均衡（具有一定程度的弹性）。

请别忘了我们也需要考虑呈指数增长的物联网(IoT),最终将包括数以亿万计的新装置，意味着设备的主机需要保持正确时间。硬件卖家无意（或有意）设置他们的设备只能与一个提供者的（甚至一个） NTP 服务器连接将成为过去，变成非常不受欢迎的问题。

你可能会想象，随着更多的硬件单元被在线购进，NTP 基础设施的拥有者大概不会为相关费用感激，因为他们正被没有实际收入所困扰。这方案远非在奇幻领域独树一帜。正当头疼 -- 感谢 NTP 通路提供的基本设置强制停止 -- 过去几年里已遇多次。

在下面两篇文章里，我将着重于一些重要的 NTP 配置和测试服务器启动。

--------------------------------------------------------------------------------

via: https://www.linux.com/learn/arrive-time-ntp-part-1-usage-overview

作者：[CHRIS BINNIE][a]
译者：[译者ID](https://github.com/XYenChi)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://www.linux.com/users/chrisbinnie
[1]:https://www.linux.com/licenses/category/used-permission
[2]:https://www.linux.com/files/images/ntp-timejpg
[3]:http://www.ntp.org/ntpfaq/NTP-s-sw-clocks-quality.htm
[4]:http://support.ntp.org/bin/view/Servers/StratumOneTimeServers