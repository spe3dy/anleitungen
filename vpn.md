````
root@vpn:~# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.1.0.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
root@vpn:~# wg-quick up wg1
[#] ip link add wg1 type wireguard
[#] wg setconf wg1 /dev/fd/63
[#] ip -4 address add 10.10.0.2/32 dev wg1
[#] ip link set mtu 1420 up dev wg1
[#] resolvconf -a tun.wg1 -m 0 -x
[#] ip -4 route add 10.1.0.3/32 dev wg1
root@vpn:~# ping 192.168.10.4
````
Ping auf die VPN - IP [OK]

````
root@vpn:~# ping 10.10.0.2
PING 10.10.0.2 (10.10.0.2) 56(84) bytes of data.
64 bytes from 10.10.0.2: icmp_seq=1 ttl=64 time=0.046 ms
64 bytes from 10.10.0.2: icmp_seq=2 ttl=64 time=0.057 ms
^C
--- 10.10.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1025ms
rtt min/avg/max/mdev = 0.046/0.051/0.057/0.005 ms
````

Ping auf einen Host hinter der VPN [nicht OK]

````
root@vpn:~# ping 192.168.10.4
PING 192.168.10.4 (192.168.10.4) 56(84) bytes of data.
^C
--- 192.168.10.4 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1024ms
````

Routing: 
````
root@vpn:~# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.31.1.1      0.0.0.0         UG    100    0        0 eth0
10.0.0.0        10.0.0.1        255.255.0.0     UG    0      0        0 enp7s0
10.0.0.1        0.0.0.0         255.255.255.255 UH    0      0        0 enp7s0
10.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 wg0
10.1.0.3        0.0.0.0         255.255.255.255 UH    0      0        0 wg1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-cf36b23dca7f
172.19.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-7f0d135ed64e
172.20.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-7a7f9924ae4b
172.24.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-06d0e8371b72
172.25.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-8b0501ac3f4b
172.26.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-84e57d1fef36
172.31.1.1      0.0.0.0         255.255.255.255 UH    100    0        0 eth0
ns1.recursivedn 172.31.1.1      255.255.255.255 UGH   100    0        0 eth0
ns2.recursivedn 172.31.1.1      255.255.255.255 UGH   100    0        0 eth0
````

Wireguadr Uebersicht:
````
root@vpn:~# wg show
interface: wg0
  public key: <mykey>
  private key: (hidden)
  listening port: 51820

peer: <myPeerkey>
  endpoint: 5.147.12.195:51821
  allowed ips: 10.1.0.2/32
  latest handshake: 1 minute, 39 seconds ago
  transfer: 411.33 KiB received, 3.92 MiB sent

peer: <peerkey>
  allowed ips: 10.1.0.3/32

peer: <peerkey>
  allowed ips: 10.1.0.4/32

interface: wg1
  public key: <myKey>
  private key: (hidden)
  listening port: 33648

peer: <peerkey>
  preshared key: (hidden)
  endpoint: 5.147.12.195:59920
  allowed ips: 10.1.0.3/32
  latest handshake: 9 seconds ago
  transfer: 736 B received, 2.50 KiB sent
  persistent keepalive: every 25 seconds
````

Verbindung von Hetzner nach Hause besteht.
![vpn-Home](http://wolke.unixbox.at/s/3rWMYRHks4SFEGX/download/2025-07-11%2021.20.12%20192.168.10.114%206eeeffc6d59a.png)

Interfaces:
````
root@vpn:~# iip
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             49.13.2.120/32 metric 100 2a01:4f8:c012:43fd::1/64 fe80::9400:2ff:fe2c:1436/64 
enp7s0           UP             10.0.0.4/32 fe80::8400:ff:fe60:b129/64 
br-7a7f9924ae4b  DOWN           172.20.0.1/16 fe80::42:8bff:fe87:8b52/64 
br-7f0d135ed64e  DOWN           172.19.0.1/16 
br-cf36b23dca7f  DOWN           172.18.0.1/16 
docker0          UP             172.17.0.1/16 fe80::42:33ff:fe42:f8bf/64 
vethf3f73e5@if15 UP             fe80::e86e:59ff:fe46:9241/64 
vethc100516@if278 UP             fe80::24f5:5fff:fe34:d6ff/64 
vethaeb8a78@if282 UP             fe80::2028:47ff:fe03:b94f/64 
br-06d0e8371b72  DOWN           172.24.0.1/16 fe80::42:e0ff:fe01:b3ed/64 
br-8b0501ac3f4b  DOWN           172.25.0.1/16 fe80::42:4cff:feca:849a/64 
veth588d351@if369 UP             fe80::4e:4bff:fe3e:c708/64 
vetha9ac56d@if391 UP             fe80::a406:d4ff:fec0:3bb8/64 
veth1d118bb@if439 UP             fe80::e83d:8ff:fe46:35ca/64 
br-84e57d1fef36  UP             172.26.0.1/16 fe80::42:2fff:fe0c:9e97/64 
veth1eac23c@if444 UP             fe80::68d4:9dff:fe8d:c94f/64 
veth449689d@if448 UP             fe80::6495:ddff:fe38:b42c/64 
wg0              UNKNOWN        10.1.0.1/24 
wg1              UNKNOWN        10.10.0.2/32 
````