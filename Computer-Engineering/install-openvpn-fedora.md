# Install Openvpn on Fedora

## Server

First, get the script from <https://github.com/angristan/openvpn-install> and make it executable:

```bash
curl -O https://raw.githubusercontent.com/angristan/openvpn-install/master/openvpn-install.sh
chmod +x openvpn-install.sh
```

Then run it:

```bash
./openvpn-install.sh
```

After that you have a vpn, but you still need to do the routing so that clients can connect to internet

```bash
sh /etc/iptables/add-openvpn-rules.sh
sysctl -w net.ipv4.ip_forward=1
```

Check the routing by

```bash
iptables -t nat -L -n -v
sysctl net.ipv4.ip_forward
```

### Add log and management feature to vpn

Edit `/etc/openvpn/server.conf `, add

```
log-append /var/log/openvpn.log
status /var/log/openvpn/status.log
management localhost 7505
```

And restart

```bash
systemctl restart openvpn-server@server.service
```

And use telnet to manage

```bash
telnet localhost 7505
```



## Add or Remove User

Just simply run the installation script again

```bash
./openvpn-install.sh
```

## Assign an Static IP for a Client

```base
echo "ifconfig-push 10.8.0.50 255.255.255.0" > /etc/openvpn/ccd/<vpn-username>
```

## Client

Download the ovpn file

```bash
scp root@vpn-server:/root/<vpn-username>.ovpn .
```

Install Openvpn

```bash
yum install openvpn
```

Connect

```bash
sudo cp desktop.ovpn /etc/openvpn/client/
sudo openvpn --client --config /etc/openvpn/client/<vpn-username>.ovpn
```

Check your connection by

```bash
dig TXT +short o-o.myaddr.l.google.com @ns1.google.com
```

Should return your vpn server's ip address

For kde users, you need to install `networkmanager-openvpn`, and add vpn through terminal:

```bash
nmcli connection import type openvpn file <vpn-username>.ovpn
```

Every time I add the vpn via kde gui, it fails. Maybe there are some bugs in importing openvpn configs.

## Troubleshooting

Linux client cannot get DNS resolution proxied: Add these to your `client.conf`

```
script-security 2
up /usr/share/openvpn/contrib/pull-resolv-conf/client.up
down /usr/share/openvpn/contrib/pull-resolv-conf/client.down
```