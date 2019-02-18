# OpenConnect VPN Server Set Up
---


## Procedures
```
apt install nginx -y
```
Please follow [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-debian-9 "DigitalOcean Community") to secure Nginx with Let's Encrypt.
> **IMPORTANT** <br>
> Please record the certificate path! Typically, you can find them in the following paths.
> ```
> /etc/letsencrypt/live/example.com/fullchain.pem
> /etc/letsencrypt/live/example.com/privkey.pem
> /etc/letsencrypt/ssl-dhparams.pem
> ```
```
service nginx stop
apt install ocserv -y
nano /etc/ocserv/ocserv.conf
```
Please find following lines in the config file, then edit or delete them.
```
auth = "plain[/etc/ocserv/ocpasswd]"
tcp-port = custom_port
udp-port = custom_port
server-cert = /etc/letsencrypt/live/example.com/fullchain.pem
server-key = /etc/letsencrypt/live/example.com/privkey.pem
dh-params = /etc/letsencrypt/ssl-dhparams.pem
# ca-cert
banner = "your_greetings"
max-clients = based_on_your_needs
max-same-clients = based_on_your_needs
try-mtu-discovery = true
# cert-user-oid
cookie-timeout = 86400
device = whatever_sounds_good
default-domain = example.com
ipv4-network = 192.168.17.0/28 #based_on_your_needs
# ipv4-netmask = 255.255.255.0
dns = 8.8.8.8
# route = 
# no-route = 
cisco-client-compat = true
```
```
nano /lib/systemd/system/ocserv.socket
```
Set `ListenStream` and `ListenDatagram` to the same port number.
```
systemctl daemon-reload
nano /etc/sysctl.conf
```
Uncomment `net.ipv4.ip_forward = 1` and `net.ipv6.conf.all.forwarding=1`. <br>
Use `ip addr` to find your Ethernet interface; Typically, it is `eth0`.
```
iptables -t nat -A POSTROUTING -o your_eth_I/F -j MASQUERADE
iptables-save > /etc/iptables.rules
nano /etc/systemd/system/iptables-restore.service
```
Please add following lines in the file.
```
[Unit]
Description=Packet Filtering Framework
Before=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/sbin/iptables-restore /etc/iptables.rules
ExecReload=/sbin/iptables-restore /etc/iptables.rules
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
```
systemctl daemon-reload
systemctl enable iptables-restore
reboot
```
Congratulations! You have finished to set up your OpenConnect VPN Server!

## Commands
1. Add or delete an user. 
```
ocpasswd -c /etc/ocserv/ocpasswd username
ocpasswd -c /etc/ocserv/ocpasswd -d username
````
2. Start, stop, and restart the OpenConnect VPN Server.
```
/etc/init.d/ocserv start
/etc/init.d/ocserv stop
/etc/init.d/ocserv restart
```
3. List existing iptables.
```
iptables -t nat -L
```
> *Attention* <br>
> Sometimes iptables rules disappear after the reboot, so you may need to add the rule again!
4. Terminate the process which uses the specific port. `kill $(lsof -t -i:port_number)`
