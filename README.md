# Surikat WireGuard VPN

VPN basada en WireGuard que permite acceder a los servicios Docker del servidor desde cualquier dispositivo conectado como peer.

## Arquitectura

```
Cliente VPN (10.8.0.2)
       │
       │  túnel WireGuard UDP:51900
       │
WireGuard container (10.8.0.1)
       │
       │  actúa como gateway NAT
       │
red Docker surikat-vpn (172.20.0.0/16)
       ├── postgres          172.20.1.2
       ├── otro-servicio     172.20.1.3
       └── ...
```

Los peers reciben rutas solo para `10.8.0.0/24` y `172.20.0.0/16` (split-tunnel). El resto del tráfico del cliente usa su conexión local normal.

---

## Levantar la VPN

### Requisitos previos

La red externa `surikat-network` debe existir:

```bash
docker network create --driver bridge --subnet 172.18.0.0/16 surikat-network
```

### Arrancar

```bash
docker compose up -d
```

Al arrancar por primera vez se generan automáticamente las configuraciones y claves de todos los peers en `./config/peer_<nombre>/`.

### Peers configurados

| Peer | Archivo de configuración |
|------|--------------------------|
| jaalvarez2818 | `config/peer_jaalvarez2818/peer_jaalvarez2818.conf` |
| alberto | `config/peer_alberto/peer_alberto.conf` |
| airam | `config/peer_airam/peer_airam.conf` |
| carlos | `config/peer_carlos/peer_carlos.conf` |
| roberto | `config/peer_roberto/peer_roberto.conf` |

Cada carpeta también contiene un **QR code** para configurar móviles directamente.

### Añadir un peer nuevo

1. Añadir el nombre a la variable `PEERS` en `docker-compose.yml`:

```yaml
- PEERS=jaalvarez2818,alberto,airam,carlos,roberto,nuevo_usuario
```

2. Reiniciar el contenedor:

```bash
docker compose up -d --force-recreate wireguard
```

La configuración del nuevo peer aparecerá en `config/peer_nuevo_usuario/`.

---

## Conectar un servicio a la VPN

Para que un servicio sea accesible desde la VPN tiene que unirse a la red `surikat-vpn` con una IP fija dentro del rango `172.20.1.x`.

### En el docker-compose del servicio

```yaml
services:
  mi_servicio:
    image: ...
    networks:
      surikat-network:          # red existente del servicio
        ipv4_address: 172.18.0.xxx
      vpn-network:              # añadir esto
        ipv4_address: 172.20.1.X  # elegir una IP libre

networks:
  surikat-network:
    external: true
  vpn-network:                  # declarar como externa
    external: true
    name: surikat-vpn
```

> No es necesario exponer puertos al host (`ports:`). Desde la VPN se accede directamente al puerto interno del contenedor.

### Ejemplo: base de datos PostgreSQL

```yaml
services:
  surikat_geoserver_db:
    build: .
    container_name: surikat_geoserver_db
    restart: always
    networks:
      surikat-network:
        ipv4_address: 172.18.0.205
      vpn-network:
        ipv4_address: 172.20.1.10

networks:
  surikat-network:
    external: true
  vpn-network:
    external: true
    name: surikat-vpn
```

Desde la VPN: `psql -h 172.20.1.10 -p 5432 -U usuario`

### IPs asignadas

Llevar un registro de las IPs usadas para evitar conflictos:

| Servicio | IP en surikat-vpn |
|----------|-------------------|
| _reservado gateway_ | 172.20.0.1 |
| surikat_geoserver_db | 172.20.1.10 |

---

## Conectarse a la VPN

### Linux / macOS

```bash
sudo wg-quick up ./config/peer_<nombre>/peer_<nombre>.conf
```

### Windows

Importar el archivo `.conf` en la aplicación oficial de WireGuard.

### Android / iOS

Escanear el QR code de `config/peer_<nombre>/peer_<nombre>.png` desde la app de WireGuard.

---

## Comandos útiles

```bash
# Ver estado de la VPN y peers conectados
docker exec surikat_wireguard wg show

# Ver logs
docker compose logs -f wireguard

# Reiniciar
docker compose restart wireguard

# Ver IP de un contenedor en surikat-vpn
docker inspect <nombre_contenedor> \
  --format '{{(index .NetworkSettings.Networks "surikat-vpn").IPAddress}}'
```
