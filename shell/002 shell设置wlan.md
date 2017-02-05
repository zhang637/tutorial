# Linux命令行设置wlan密码
## 查看网卡状态
	iwconfig
	
```
lo        no wireless extensions.

eth0      no wireless extensions.

wlan0     IEEE 802.11bgn  ESSID:"huitongkeji"  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=0 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:on
```

## 激活网卡
	ifconfig wlan0 up
## 扫描网络
	iwlist wlan0 scan

```
Cell 11 - Address: FC:B6:98:34:3C:C0
                    Channel:11
                    Frequency:2.462 GHz (Channel 11)
                    Quality=27/70  Signal level=-83 dBm  
                    Encryption key:on
                    ESSID:"huitongkeji"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=000000000f08729f
                    Extra: Last beacon: 132ms ago
                    IE: Unknown: 000B687569746F6E676B656A69
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 03010B
                    IE: Unknown: 050400010000
                    IE: Unknown: 0706555320010B1E
                    IE: Unknown: 2A0102
                    IE: Unknown: 32043048606C
                    IE: Unknown: 2D1AAD0103FFFF0000000000000000000001000000000406E6A70C00
                    IE: Unknown: 3D160B000500000000000000000000000000000000000000
                    IE: Unknown: 7F080000000000000040
                    IE: Unknown: DD180050F2020101800003A4000027A4000042435E0062322F00
                    IE: Unknown: DD0900037F01010000FF7F
                    IE: IEEE 802.11i/WPA2 Version 1
                        Group Cipher : TKIP
                        Pairwise Ciphers (2) : CCMP TKIP
                        Authentication Suites (1) : PSK
                    IE: WPA Version 1
                        Group Cipher : TKIP
                        Pairwise Ciphers (2) : CCMP TKIP
                        Authentication Suites (1) : PSK
```
发现WPA2-PSK/WPA-PSK 加密网络 huitongkeji

因为加密方式为 WPA-PSK 所以得用 wpa_supplicant 而不能使用 iwconfig wlan0 key xxx 形式，iwconfig key方式适用于WEP。
### 安装wpa_supplicant
	yum install wpa_supplicant
### 配置wpa_supplicant
编辑文件/etc/wpa_supplicant/wpa_supplicant.conf
### 连接wlan0到网络，并以daemon方式运行
	wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
-B Background 在后台以daemon 运行
-i interface 
-c 配置文件
