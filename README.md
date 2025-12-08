# Reglas de enrutamiento hacia la red docker

Copy to config/wg_confs/wg0.conf

[Interface]
Address = 10.8.0.1
ListenPort = 51820
PrivateKey = uEqEI7f/h/5ydNTNrlZdyxqBnaVTZrcvq8kNe6DCI3w=
#PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE
#PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE; iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE; iptables -t nat -A PREROUTING -i %i -p tcp --dport 9443 -j DNAT --to-destination 172.18.0.24:9443; iptables -t nat -A POSTROUTING -o eth+ -j MASQUERADE; iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE; iptables -t nat -A PREROUTING -i %i -p tcp --dport 8443 -j DNAT --to-destination 172.18.0.30:443; iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE; iptables -t nat -A PREROUTING -i %i -p tcp --dport 5433 -j DNAT --to-destination 172.18.0.25:5432
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE; iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE; iptables -t nat -D PREROUTING -i %i -p tcp --dport 9443 -j DNAT --to-destination 172.18.0.24:9443; iptables -t nat -D POSTROUTING -o eth+ -j MASQUERADE; iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE; iptables -t nat -D PREROUTING -i %i -p tcp --dport 8443 -j DNAT --to-destination 172.18.0.30:443; iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -d 172.18.0.0/16 -j MASQUERADE; iptables -t nat -D PREROUTING -i %i -p tcp --dport 5433 -j DNAT --to-destination 172.18.0.25:5432
