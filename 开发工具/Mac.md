# Mac 相关配置
## 1. 实现内外网双通
- 通过设置路由实现内外网双通

```shell
# 第一步 : 查看路由表 : netstat -nr
cy@mac  ~  netstat -nr
# 以下为路由列表, 其实往下还有,只是下边的没必要看,只用看 default 就行了. 此处为已经设置好的
Routing tables

Internet:
Destination        Gateway            Flags        Refs      Use   Netif Expire
# 默认外网路由 @@@
default            172.20.10.1        UGSc           98        0     en7
# 默认内网路由 @@@
default            18.141.73.254      UGScI           5        0     en8
18.141.73/24       link#20            UCS             8        0     en8      !
18.141.73.2        40:62:31:3:7a:32   UHLWI           0       28     en8   1195

# 第二步 : 查看默认路由 : route get 0.0.0.0
cy@mac  ~  route get 0.0.0.0
   route to: default
destination: default
       mask: default
    gateway: localhost
  interface: en7
      flags: <UP,GATEWAY,DONE,STATIC,PRCLONING>
 recvpipe  sendpipe  ssthresh  rtt,msec    rttvar  hopcount      mtu     expire
       0         0         0         0         0         0      1500         0
       
# 第三步 : 删除默认路由
sudo route delete 0.0.0.0

# 第四步 : 添加外网网关
sudo route add -net 0.0.0.0 172.20.1.1

# 第五步 : 添加内网网关
sudo route add -net 166.0.0.0 166.6.66.254(就是系统设置内 网络配置内配置的 路由器)

# 第六步 : 设置网络服务顺序
系统设置 -> 网络设置 -> 左边栏下边的设置按钮 -> 设定服务顺序 -> 将外网(WiFi 或者手机 USB) 拉到第一位
```