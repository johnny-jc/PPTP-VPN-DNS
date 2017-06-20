# PPTP-VPN-DNS

安装pptp
rpm -i http://poptop.sourceforge.net/yum/stable/rhel6/pptp-release-current.noarch.rpm
yum -y install pptpd
配置dncpIP
vim /etc/pptpd.conf
localip 192.168.1.3 # 服务器IP
remoteip 192.168.1.4-192.168.1.253 # dhcpIP段
配置dns
vim /etc/ppp/pptpd-options.pptpd
ms-dns 192.168.1.233 # 内部dns 用于访问内部域名地址
配置账号 密码
vim /etc/ppp/chap-secrets
账号，协议，密码，ip地址
kevin   *   123456  *
kevin   pptpd   123456  *
开机启动
service pptpd restart
chkconfig pptpd on
开启IP转发 nat映射
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to 192.168.3

# 解决nat转发上网慢
iptables -A FORWARD -p tcp --syn -s 192.168.1.0/24 -j TCPMSS --set-mss 1356
# 防火墙开启1723端口
iptables -A INPUT -p tcp --dport 1723 -j ACCEPT
# 如果在内网 需要配置网关转发
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 1723 -j DNAT --to SERVER_IP
iptables -t nat -A PREROUTING -i eth0 -p 47 -j DNAT --to SERVER_IP
# 配置mtu
vim /etc/ppp/ip-up # exit 0 之前
/sbin/ifconfig $1 mtu 1500 #查看相应网卡获取mtu值
