# Reglas de enrutamiento hacia la red docker

Copy to ```config/wg_confs/wg0.conf```

```
[Interface]
Address = 10.8.0.1
ListenPort = 51820
PrivateKey = uEqEI7f/h/5ydNTNrlZdyxqBnaVTZrcvq8kNe6DCI3w=
#PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
#PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE

PostUp = iptables -A FORWARD -i %i -j ACCEPT
PostUp = iptables -A FORWARD -o %i -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
PostUp = iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE
PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 9443 -j DNAT --to-destination 172.18.0.200:9443
PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 8081 -j DNAT --to-destination 172.18.0.201:3000
PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 8082 -j DNAT --to-destination 172.18.0.202:80
PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 8083 -j DNAT --to-destination 172.18.0.203:80
PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 8084 -j DNAT --to-destination 172.18.0.204:80
PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 8443 -j DNAT --to-destination 172.18.0.30:443
PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 5433 -j DNAT --to-destination 172.18.0.25:5432

PostDown = iptables -D FORWARD -i %i -j ACCEPT
PostDown = iptables -D FORWARD -o %i -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 9443 -j DNAT --to-destination 172.18.0.200:9443
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 8081 -j DNAT --to-destination 172.18.0.201:3000
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 8082 -j DNAT --to-destination 172.18.0.202:80
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 8083 -j DNAT --to-destination 172.18.0.203:80
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 8084 -j DNAT --to-destination 172.18.0.204:80
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 8443 -j DNAT --to-destination 172.18.0.30:443
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 5433 -j DNAT --to-destination 172.18.0.25:5432
```
