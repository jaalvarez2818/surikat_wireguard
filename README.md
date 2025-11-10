# Reglas de enrutamiento hacia la red docker

Copy to config/wg_confs/wg0.conf

PostUp = iptables -A FORWARD -i %i -o br-72a4aa10a2e5 -j ACCEPT
PostUp = iptables -A FORWARD -i br-72a4aa10a2e5 -o %i -j ACCEPT
PostUp = iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o br-72a4aa10a2e5 -j MASQUERADE

PostDown = iptables -D FORWARD -i %i -o br-72a4aa10a2e5 -j ACCEPT
PostDown = iptables -D FORWARD -i br-72a4aa10a2e5 -o %i -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -o br-72a4aa10a2e5 -j MASQUERADE

PostUp = iptables -t nat -A PREROUTING -i %i -p tcp --dport 5433 -j DNAT --to-destination 172.17.0.1:5433
PostDown = iptables -t nat -D PREROUTING -i %i -p tcp --dport 5433 -j DNAT --to-destination 172.17.0.1:5433
