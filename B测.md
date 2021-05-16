# B测

## 单臂路由1.2.5.8

https://blog.csdn.net/cczxsong/article/details/90779833

1.交换机划分vlan2，3

2.每个端口分别在不同vlan下

```
# 划分vlan
Switch(config)#vlan 2    
Switch(config-vlan)#exit  
Switch(config)#vlan 3    
Switch(config-vlan)#exit
# 将端口分到对应的vlan
Switch(config)#interface fa 0/23   
Switch(config-if)#switch access vlan 2 
Switch(config-if)#exit

Switch(config)#interface fa 0/24      
Switch(config-if)#switch access vlan 3
Switch(config-if)#exit
# 设置交换机与路由的端口
Switch(config)#interface fastEthernet 0/1
Switch(config-if)#switchport mode trunk　　# 配置接口为trunk口，trunk口允许多个VLAN通过。
Switch(config-if)#switchport trunk allowed vlan 2,3　　# 配置turnk口允许vlan2 和vlan3通过，除了vlan2和vlan3，其他所有的vlan默认拒绝所有
```

3.设置pc的ip

**设置pc机的相应的ip和gateway**

4.连接路由，设置路由的静态IP

```
Switch>enable　　# 进入到特权模式，该模式下的权限只允许查看命令
Swtich#config terminal　　# 进入到全局模式，
# 该模式下的权限可以配置任何命令，如果想要执行查询命令，只需要在命令行首部加上 do命令，后面接需要查询的命令
Switch(config)#hostname R4　　# 更改路由器的为R1
R4(config)#interface FastEthernet 0/0　　# 进入到接口0/0
R4(config-if)#no shutdown　　# 启动接口，路由器的接口状态默认是shutdown
R4(config-if)#exit　　# 退出该接口
R4(config)#interface fastEthernet0/0.1　　
# 进入到F0/0.1接口
R4(config-subif)#encapsulation dot1Q 2 　　
# 将vlan2封装在F0/0.1接口
R4(config-subif)#ip address 192.168.2.1 255.255.255.0　　
# 配置接口的ip地址，该ip地址作为vlan 2内的电脑的网关
R4(config-subif)#exit　　
# 退出F0/0.1接口
R4(config)#interface fastEthernet 0/0.2　　
# 进入到F0/0.1接口
R4(config-subif)#encapsulation dot1Q 3　　
# 将vlan3封装在F0/0.2接口
R4(config-subif)#ip address 192.168.3.1 255.255.255.0　　
# 配置ip地址，该ip地址作为vlan3内的电脑的网关
```

5.设置路由的端口ip（用于跳转）

非有子接口的端口IP，作为路由本身IP

设置静态路由，目的network，mask，经过的next hop

6.总共五个单臂路由，如上所述设置。先是设置五个单臂路由，然后通过单臂路由和静态路由的设置，使得五个单臂路由连通，任意一个pc机均可通信

7.设置完每个路由器的静态路由，network目的，经过路由ip

8.可以全部ping通

### dhcp3

1.进入子接口中

2.ip dhcp pool 1

3.network 192.168.20.0 255.255.255.0

4.default-router 192.168.20.1

5.dns-server 144.144.144.144

两个子接口一样设置，ip，网关不同

### nat4

[nat配置]: https://blog.51cto.com/u_13685543/2145838

1.设置服务器和pc的ip和网关

2.设置路由的ip和网关

3.激活路由端口，并分别设置为inside和outside

```
ip nat inside/outside
duplex auto
speed auto
```

4.

inside source：初始于 inside 向 outside 发送，在outside接口执行源地址翻译。

将内部局部地址转换为内部全局地址;数据方向inside->outside,在outside上执行转换;

**静态 NAT** ( Static NAT )（ ***\*一对一\**** ）。将内部网络的私有IP地址转换为公有IP地址，IP地址对是一对一的，是一直不变的。![img](https://img-blog.csdn.net/20170913142720836)

```
ip nat inside source static server-ip pc-ip
ip nat inside source static local-address global-address
```

5.设置静态路由（转发）

### gre6

tracert 192.168.47.2 用来记录路径

R8

```
Router>en
Router#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#interface tunnel 10

Router(config-if)#
%LINK-5-CHANGED: Interface Tunnel10, changed state to up

Router(config-if)#ip add 172.16.1.2 255.255.255.0
Router(config-if)#tunnel source fa0/0
Router(config-if)#tunnel destination 192.168.45.66
Router(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel10, changed state to up

Router(config-if)#no shutdown
Router(config-if)#exit
Router(config)#ip route 192.168.45.0 255.255.255.240 172.16.1.1
```

```
R4

Router>en
Router#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#interface tunnel 10

Router(config-if)#
%LINK-5-CHANGED: Interface Tunnel10, changed state to up

Router(config-if)#ip add 172.16.1.2 255.255.255.0
Router(config-if)#tunnel source fa0/0
Router(config-if)#tunnel destination 192.168.45.66
Router(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface Tunnel10, changed state to up

Router(config-if)#no shutdown
Router(config-if)#exit
Router(config)#ip route 192.168.45.0 255.255.255.240 172.16.1.1


```



### acl7

```
R8 配置ACL
access-list 1 deny host 192.168.45.34  拒绝 192.168.45.34 访问
access-list 1 permit any 其他的都允许访问
int fa0/0  进入fa0/0端口
ip access-group 1 in  讲规则1应用于过滤fa0/0j接受的报文
exit
```

