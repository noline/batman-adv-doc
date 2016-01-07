.. batman-adv文档 documentation master file, created by
   sphinx-quickstart on Thu Jan 07 15:19:19 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


Batman-adv中文文档
==========================================

Contents:

.. toctree::
   :maxdepth: 2

一、batman相关配置修改
==========================================
batman-adv将所有已配置好的接口虚拟成一个单mesh云接口,允许一个接口同时存在于多个云接口中。在batman-adv启动之后，会扫描可用的接口，一旦找到，会建立一个子文件夹在/sys/class/net/<iface>下：
::

   ls /sys/class/net/eth0/batman_adv/
   iface_status  mesh_iface

将一个接口加入mesh云接口只需要写入相应文件即可：
::

    echo bat0 > /sys/class/net/eth0/batman_adv/mesh_iface

停用一个节点：
::

    echo none > /sys/class/net/eth0/batman_adv/mesh_iface

在/sys/class/net/bat0/mesh/下可以看到每个云接口的信息：
::

    ls  /sys/class/net/bat0/mesh/
    aggregated_ogms        gw_bandwidth           multicast_mode
    ap_isolation           gw_mode                network_coding
    bonding                gw_sel_class           orig_interval
    bridge_loop_avoidance  hop_penalty            routing_algo
    distributed_arp_table  isolation_mark
    fragmentation          log_level

在batman协议中会通过将控制消息整合成一个包发送来减少协议负载，这个措施被默认实现。如果在高度移动的环境下，可能需要关闭该功能，可以通过下述方法关闭：
::

    cat /sys/class/net/bat0/mesh/aggregated_ogms
    enabled

默认运行模式为 "interface alternating"，可以在特殊一跳的情况下使用bonding模式：
::

    cat /sys/class/net/bat0/mesh/bonding 
    disabled

在2014.1.0版本之后，跳数惩罚改变了形式：在OGM包在与之前接收接口不一样的接口上发送时，使用一次跳数惩罚，在同一接口发送时使用两次。这个被用来处理半双工路由，使得路由在有一条相似路径时更倾向于切换接口。可以通过以下方式来更改惩罚值：
::

    cat /sys/class/net/bat0/mesh/hop_penalty 
    15

在比较稳定的环境下可以提高发送间隔来减少负载。更改发送间隔的方式如下：
::

    cat /sys/class/net/bat0/mesh/orig_interval 
    1000




二、batman-adv协议相关概念
==============================================
由于ad-hoc网络的特殊性，此种网络不具有固定的结构，网络拓扑动态变化，并且基于不可靠的固有的介质（are based on an inherently unreliable medium），使得传统协议不再适用。

OLSR协议是如今使用最为广泛的ad-hoc网络协议，由于城域mesh网的发展，OLSR已经在原版的基础上做出了很大的变化。在MPR机制与迟滞现象（Hysterese）在现实网络中显得不再适用的时候，OLSR加入了鱼眼与ETX机制。然而，随着Mesh网的网络规模逐渐扩大，并且所有的链路状态路由协议都不可避免的要重新计算全网拓扑，对资源消耗较大，在小型的嵌入式系统中会造成较大问题。经测算，在一片小型嵌入式芯片中，当节点规模为450时，重新计算全网拓扑时间达到数秒。

Batman-adv协议致力于将最佳端到端路径问题从Mesh网中的节点分割成所有实际组网节点。所有节点仅仅感知并存储到到所有其他节点的最优下一跳信息。这样，全网拓扑的感知与计算问题就得到解决。此外，一种事件驱动并且无超时（无超时是指batman-adv不会为拓扑信息设置时效或周期更新来优化路由决策）的洪泛机制避免了拓扑信息的无限增长和减少mesh网中洪泛拓扑信息的数量（减少了链路负载）。该算法的设计用于处理链路不稳定的网络情况。

简述一下batman-adv的运行机制。每个节点传输广播包（OGM包）来向其邻居节点告知它的存在。然后它的邻居也将这些OGM包通过一些特殊机制重新广播来告知它们各自的邻居，这样两跳邻居也能感知到该节点的存在。这样，随着这些OGM包被不断转发，更多节点能感知到该节点的存在。OGM不叫小，传统数据包在包含UDP和IP报头的时候大约是52byte，OGM至少包含源节点地址，转发节点地址，TTL和段号。

OGM在质量较差或不稳定的链路上传输时，会出现丢包和延时的情况。因此，通过较好的链路传输时会更快，更稳定。为了标识OGM被收到一次或多次，OGM包在从源端发出来时携带有段号，每个节点最多只转发一个OGM一次，并且只接收最早到达和最稳定的OGM包，转发该包的邻居节点被认定为到源节点的最佳下一跳。


三、路由协议应该能处理的情况
================================================


四、batman-adv重要数据结构
================================================

