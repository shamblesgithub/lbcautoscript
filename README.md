#!/bin/bash
clear
echo "                                                                 "
echo "                         LBC SETUP                               "
echo "                      Made by: Shambles                          "
echo "                        AUTO SCRIPT                              "
echo "                                                                 "
echo " "
 
HOST=""
USER=""
HUB=""
BRIDGE=""
 
 
HOST=${HOST}
HUB=${HUB}
BRIDGE=${BRIDGE}
USER=${USER}
 
echo -n "Enter Server IP: "
read HOST
echo -n "Set Virtual Hub: "
read HUB
echo -n "Set ${HUB} hub username: "
read USER
echo -n "Set ${BRIDGE} bridge name: "
read BRIDGE
echo " "
echo "Now sit back and wait until the installation finished."
echo " "
yum update
yum -y groupinstall "Development Tools"
yum -y install gcc zlib-devel openssl-devel readline-devel ncurses-devel wget tar dnsmasq net-tools iptables-services system-config-firewall-tui nano iptables-services firewalld systemd
systemctl disable firewalld
systemctl stop firewalld
systemctl status firewalld
service iptables save
service iptables stop
chkconfig iptables off
wget http://www.softether-download.com/files/softether/v4.27-9666-beta-2018.04.21-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.27-9666-beta-2018.04.21-linux-x64-64bit.tar.gz
tar -xzf softether-vpnserver-v4.27-9666-beta-2018.04.21-linux-x64-64bit.tar.gz
cd vpnserver
sudo make
cd ..
mv vpnserver /usr/local
cd /usr/local/vpnserver/
chmod 600 *
chmod 700 vpncmd
chmod 700 vpnserver
 
echo '#!/bin/sh
# chkconfig: 2345 99 01
# description: SoftEther VPN Server
DAEMON=/usr/local/vpnserver/vpnserver
LOCK=/var/lock/subsys/vpnserver
TAP_ADDR=10.30.20.1
TAP_ADAPTER='tap_soft'
 
test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
sleep 3
sleep 1
ifconfig $TAP_ADAPTER $TAP_ADDR
service dnsmasq start
;;
stop)
$DAEMON stop
rm $LOCK
;;
restart)
$DAEMON stop
sleep 3
$DAEMON start
sleep 1
ifconfig $TAP_ADAPTER $TAP_ADDR
service dnsmasq restart
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0' > /etc/init.d/vpnserver
chmod +x /etc/init.d/vpnserver
/etc/init.d/vpnserver start
systemctl enable vpnserver
/usr/local/vpnserver/vpncmd
ServerPasswordSet
HOST=${HOST}
HUB=${HUB}
USER=${USER}
BRIDGE=${BRIDGE}
HubCreate ${HUB}
BridgeCreate /DEVICE:"soft" /TAP:yes ${BRIDGE}
VpnOverIcmpDnsEnable /ICMP:yes /DNS:yes
Hub ${HUB}
UserCreate ${USER}
UserAnonymousSet ${USER}
exit
cd ..
HOST=${HOST}
HUB=${HUB}
USER=${USER}
BRIDGE=${BRIDGE}
service vpnserver stop
echo interface=tap_soft >> /etc/dnsmasq.conf
echo dhcp-range=tap_soft,10.30.20.5,10.30.20.50,12h >> /etc/dnsmasq.conf
echo dhcp-option=tap_soft,3,10.30.20.1 >> /etc/dnsmasq.conf
echo port=0 >> /etc/dnsmasq.conf
echo dhcp-option=option:dns-server,208.67.222.222,208.67.220.220 >> /etc/dnsmasq.conf
echo 'net.ipv4.ip_forward = 1' > /etc/sysctl.d/ipv4_forwarding.conf
cat /etc/sysctl.d/ipv4_forwarding.conf
sysctl --system
service dnsmasq restart && service vpnserver restart
yum install bind-utils
iptables -A INPUT -i eth0 -p tcp --destination-port 53 -s ${HOST} -j DROP
iptables -A INPUT -i eth0 -p tcp --destination-port 443 -s ${HOST} -j DROP
iptables -A INPUT -i eth0 -p tcp --destination-port 80 -s ${HOST} -j DROP
iptables -A INPUT -i eth0 -p tcp --destination-port 22 -s ${HOST} -j DROP
iptables -t nat -A POSTROUTING -s 10.30.20.0/24 -j SNAT --to-source ${HOST}
/etc/init.d/vpnserver restart
echo "Softether server configuration has been done!"
echo " "
echo "Host: ${HOST}"
echo "Virtual Hub: ${HUB}"
echo "Port: 443, 53, 137"
echo "Username: ${USER}"
echo "Bridgename: ${BRIDGE}"
echo " "
