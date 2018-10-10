[原文地址](http://www.linuxdiyf.com/linux/29760.html)

# ubuntu server 16.10无线网卡没有启用，找不到wlan0端口

最近在安装ubuntu server最新版本时，发现无法联网，用ifconfig -a 命令查询时，发现无wlan端口，搜索后下面这篇文章能解决问题，特此整理记录下。
 
Ubuntu Server默认的情况下是不会启用无线网卡的，想想实际服务器上怎么可能有无线网卡呢，呵呵。所以我们需要手动来启用无线网卡，难点就在这里了。

使用ifconfig命令，发现没有wlan口，但是使用：
```ssh
sudo lshw -numeric -class network
sudo ifconfig -a
```

就可以找到wlan0口，其实ubuntu server默认不启用无线网卡，只需要改一下配置文件启用无线网卡就可以了。
 
1. 安装wpasupplicant。由于Ubuntu 10.04 Server已经集成了这个包，所以无需安装。如果是其他版本的Ubuntu Server，可以使用下面的命令进行安装：
```ssh
#apt-get install wpasupplicant
```
2. 生成无线路由密钥。这一步就是根据你无线网络的SSID和密码，来生成WLAN需要的配置文件。命令如下：
```ssh
#wpa_passphrase 无线网络SSID 无线网络密码 > 配置文件名
```
比如你的无线网络SSID是TP-LINK，密码是123456，生成的配置文件名为/etc/wpa_config.conf，就这样输入：
```ssh
#wpa_passphrase TP-LINK 123456 > /etc/wpa_config.conf
```
注意后面的/etc/wpa_config.conf文件名可以随意取，但是请注意不要有重名的情况产生。
 
3. 设置无线网络。编辑/etc/network/interfaces文件，将wlan添加到其中：
```ssh
#vim /etc/network/interfaces
```
在里面加上：
```ssh
auto wlan0
iface wlan0 inet dhcp
wpa-conf /etc/wpa_config.conf
```
注意如果你的路由器没有开启DHCP，则需要手动配置address、netmask、gateway、network和broadcast几项参数，这里不多说。另外就是wpa-conf后面跟着你刚才产生的密钥配置文件名。
如果一直不需要使用有线网络，可以连有线网络一起禁用掉，将auto eth0注释掉即可。
 
4. 重新启动计算机。根据我实际的操作结果来看，配置好了之后虽然说无线网卡被启用了，但是驱动貌似没加载全。因此需要重启Ubuntu Server以便完整启用无线网卡。
 
至此，Ubuntu Server也可以用无线网卡连接到无线路由器上网了。
