# 知识总结
## 网络
### 网关
	ifconfig
```
wlan0     Link encap:Ethernet  HWaddr 60:57:18:D7:F5:EB  
          inet addr:192.168.108.173  Bcast:192.168.109.255  Mask:255.255.254.0
          inet6 addr: fe80::6257:18ff:fed7:f5eb/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:12277 errors:0 dropped:0 overruns:0 frame:0
          TX packets:253 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:860976 (840.7 KiB)  TX bytes:47570 (46.4 KiB)
```
掩码：255.255.254.0
广播：192.168.109.255

```
[root@zylhdp ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.22.0    *               255.255.255.0   U     0      0        0 eth0
192.168.108.0   *               255.255.254.0   U     0      0        0 wlan0
link-local      *               255.255.0.0     U     1002   0        0 eth0
link-local      *               255.255.0.0     U     1003   0        0 wlan0
default         192.168.109.1   0.0.0.0         UG    0      0        0 wlan0
[root@zylhdp ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
192.168.22.0    0.0.0.0         255.255.255.0   U         0 0          0 eth0
192.168.108.0   0.0.0.0         255.255.254.0   U         0 0          0 wlan0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 wlan0
0.0.0.0         192.168.109.1   0.0.0.0         UG        0 0          0 wlan0
```
可知Gateway：192.168.109.1

