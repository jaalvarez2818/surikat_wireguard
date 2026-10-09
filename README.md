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
       ├── red surikat-network (172.18.0.0/16)   servicios existentes
       │       ├── surikat_geoserver_db   172.18.0.205
       │       └── ...
       │
       └── red surikat-vpn (172.20.0.0/16)       servicios fuera de surikat-network
               └── ...
```

Los peers reciben rutas solo para `10.8.0.0/24`, `172.18.0.0/16` y `172.20.0.0/16` (split-tunnel). El resto del tráfico del cliente usa su conexión local normal.

> Si se cambia `ALLOWEDIPS` (o cualquier variable de entorno), hay que recrear el contenedor y **volver a importar** la configuración en cada cliente, ya que los `.conf` antiguos conservan las rutas anteriores.

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

## Acceder a los servicios desde la VPN

Desde la VPN se accede directamente a la **IP interna del contenedor** y a su **puerto interno** (no al puerto mapeado en el host). No es necesario exponer puertos (`ports:`). Los nombres de contenedor no se resuelven por DNS: hay que usar IPs.

### Servicios en surikat-network

Cualquier contenedor en `surikat-network` es accesible sin cambios. Conviene que tenga IP fija para que no cambie al reiniciar:

```yaml
services:
  surikat_geoserver_db:
    build: .
    container_name: surikat_geoserver_db
    restart: always
    networks:
      surikat-network:
        ipv4_address: 172.18.0.205

networks:
  surikat-network:
    external: true
```

Desde la VPN: `psql -h 172.18.0.205 -p 5432 -U usuario`

### Servicios fuera de surikat-network

Si un servicio no está en `surikat-network`, se puede unir a la red `surikat-vpn` con una IP fija en el rango `172.20.1.x`:

```yaml
services:
  mi_servicio:
    image: ...
    networks:
      vpn-network:
        ipv4_address: 172.20.1.X  # elegir una IP libre

networks:
  vpn-network:
    external: true
    name: surikat-vpn
```

### IPs asignadas en surikat-vpn

Llevar un registro de las IPs usadas para evitar conflictos:

| Servicio | IP en surikat-vpn |
|----------|-------------------|
| _reservado gateway_ | 172.20.0.1 |

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

# Ver IP de un contenedor (sustituir la red por surikat-vpn si aplica)
docker inspect <nombre_contenedor> \
  --format '{{(index .NetworkSettings.Networks "surikat-network").IPAddress}}'
```

### Si conecta pero no se llega a los servicios

1. `docker exec surikat_wireguard wg show` → el peer debe tener un `latest handshake` reciente. Si no, revisar que el firewall (GCP) permite **UDP 51900** de entrada.
2. Comprobar que el `.conf` del cliente tiene en `AllowedIPs` la red del servicio. Si no, reimportarlo.
3. Comprobar que la red local del cliente no usa `172.18.x.x` ni `172.20.x.x` (conflicto de rutas).
