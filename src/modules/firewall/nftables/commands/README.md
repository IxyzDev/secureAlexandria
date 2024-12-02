## Guía Completa de `nftables` (de Básico a Avanzado)

### 1. **Introducción a `nftables`**
`nftables` es una herramienta de firewall que reemplaza a `iptables` en sistemas Linux modernos. Permite definir reglas de filtrado de paquetes, NAT, y otras configuraciones relacionadas con la seguridad de la red.

### 2. **Estructura Básica de un Comando de `nftables`**
La estructura básica de los comandos de `nftables` es la siguiente:

```bash
nft [opción] <subcomando> <comando> <tabla> <cadenas> <acciones>
```

**Explicación de los elementos:**
- `nft`: El comando principal para interactuar con `nftables`.
- `[opción]`: Opciones adicionales para modificar la salida o comportamiento del comando.
- `<subcomando>`: Acciones principales como `add`, `list`, `delete`, etc.
- `<comando>`: Acciones específicas a realizar como `rule`, `table`, `chain`, etc.
- `<tabla>`: El espacio donde se definirán las reglas, por ejemplo, `filter`, `nat`, etc.
- `<cadenas>`: Las cadenas dentro de la tabla donde se colocan las reglas, como `INPUT`, `OUTPUT`, `FORWARD`.
- `<acciones>`: Las acciones a ejecutar, como aceptar o rechazar tráfico (`accept`, `drop`, `reject`).

Estas opciones permiten modificar el comportamiento del comando `nft`. Aquí te muestro las opciones más comunes:

| **Opción**         | **Descripción**                                                                   |
| ------------------ | --------------------------------------------------------------------------------- |
| `-h` o `--help`    | Muestra la ayuda del comando y una breve descripción de las opciones disponibles. |
| `-v` o `--verbose` | Muestra más detalles sobre las reglas, generalmente en formato más descriptivo.   |
| `-n` o `--numeric` | Muestra direcciones IP y puertos en formato numérico en lugar de traducirlos.     |
| `-f <archivo>`     | Permite cargar reglas desde un archivo.                                           |
| `-i <interfaz>`    | Especifica la interfaz para aplicar la regla (por ejemplo, `eth0`, `wlan0`).      |
| `-s <ip>`          | Especifica una dirección IP de origen en algunas operaciones.                     |

### 2. **Subcomandos de `nftables`**
`nftables` usa diferentes subcomandos para realizar tareas específicas. A continuación se detallan los más comunes:

| **Subcomando** | **Descripción**                                                                                |
| -------------- | ---------------------------------------------------------------------------------------------- |
| `add`          | Añadir nuevas reglas, tablas, cadenas, o elementos.                                            |
| `list`         | Mostrar las reglas, tablas, cadenas, o el conjunto completo de reglas (`ruleset`).             |
| `delete`       | Eliminar reglas, tablas, cadenas, o elementos específicos.                                     |
| `flush`        | Vaciar (eliminar todas) las reglas dentro de una cadena específica.                            |
| `replace`      | Reemplaza una tabla, cadena o regla por otra (usualmente para evitar errores por duplicación). |
| `update`       | Actualiza reglas, cadenas o tablas sin eliminarlas, lo cual permite modificaciones en vivo.    |

### 3. **Comandos de `nftables`**
Los comandos principales de `nftables` son los siguientes:

| **Comando** | **Descripción**                                                         |
| ----------- | ----------------------------------------------------------------------- |
| `table`     | Crea, lista o elimina tablas.                                           |
| `chain`     | Crea, lista, elimina o modifica cadenas dentro de una tabla.            |
| `rule`      | Crea, lista, elimina o modifica reglas dentro de una cadena específica. |
| `set`       | Crea y administra conjuntos de direcciones IP, puertos o protocolos.    |

### 4. **Tablas en `nftables`**
Las tablas son el espacio donde se almacenan las cadenas y reglas. En `nftables`, hay diferentes tipos de tablas que se usan dependiendo de lo que quieras hacer:

| **Tipo de Tabla** | **Descripción**                                                                 |
| ----------------- | ------------------------------------------------------------------------------- |
| `inet`            | Soporta IPv4 e IPv6. Es una tabla común para la mayoría de las configuraciones. |
| `ip`              | Específica para IPv4.                                                           |
| `ip6`             | Específica para IPv6.                                                           |
| `bridge`          | Utilizada para configuraciones en redes de puente (bridge).                     |
| `nat`             | Para configurar Network Address Translation (NAT).                              |
| `filter`          | Usada para el filtrado de paquetes (Firewall).                                  |

### 5. **Cadenas en `nftables`**
Las cadenas son donde se colocan las reglas. Cada tabla puede tener varias cadenas, y cada cadena se asocia con un *hook* (enganche) específico. Los hooks determinan en qué punto del procesamiento de paquetes se aplica la cadena (por ejemplo, al recibir tráfico o al enviarlo).

| **Cadena**    | **Descripción**                                                              | **Hook**                          |
| ------------- | ---------------------------------------------------------------------------- | --------------------------------- |
| `input`       | Para filtrar paquetes de entrada hacia el sistema.                           | `input` (entrada)                 |
| `output`      | Para filtrar paquetes salientes del sistema.                                 | `output` (salida)                 |
| `forward`     | Para filtrar paquetes que atraviesan el sistema (por ejemplo, en un router). | `forward` (reenviar)              |
| `prerouting`  | Para modificar paquetes antes de ser enrutados (por ejemplo, en DNAT).       | `prerouting` (pre-enrutamiento)   |
| `postrouting` | Para modificar paquetes después de ser enrutados (por ejemplo, en SNAT).     | `postrouting` (post-enrutamiento) |

### 6. **Métodos de Reglas y Acciones**
Las reglas en `nftables` definen cómo se procesan los paquetes. Estas reglas pueden estar basadas en direcciones IP, puertos, protocolos, etc. Aquí te explico los métodos y acciones más comunes:

#### 6.1 **Métodos en las Reglas**
En `nftables`, los métodos permiten filtrar el tráfico según diversos parámetros como direcciones, puertos, o protocolos.

| **Método**    | **Descripción**                                                    |
| ------------- | ------------------------------------------------------------------ |
| `ip saddr`    | Filtra por dirección IP de origen.                                 |
| `ip daddr`    | Filtra por dirección IP de destino.                                |
| `tcp dport`   | Filtra por el puerto TCP de destino.                               |
| `udp sport`   | Filtra por el puerto UDP de origen.                                |
| `ip protocol` | Filtra por el protocolo (por ejemplo, `tcp`, `udp`, `icmp`, etc.). |
| `mac source`  | Filtra por la dirección MAC de origen.                             |

#### 6.2 **Acciones en las Reglas**
Las acciones definen qué hacer con el paquete una vez que se ha pasado la regla de filtrado.

| **Acción**   | **Descripción**                                                                   |
| ------------ | --------------------------------------------------------------------------------- |
| `accept`     | Acepta el paquete.                                                                |
| `drop`       | Descarta el paquete sin notificar al remitente.                                   |
| `reject`     | Descarta el paquete y envía un mensaje de error al remitente (por ejemplo, ICMP). |
| `masquerade` | Realiza NAT de salida (modifica la IP de origen).                                 |
| `redirect`   | Redirige el tráfico a otro puerto o dirección.                                    |

#### 6.3 **Configuración de Sets**
Un conjunto (set) es un contenedor eficiente para múltiples direcciones o puertos, lo que permite aplicar reglas de manera más eficiente cuando se manejan listas largas.

| **Comando**   | **Descripción**                                            |
| ------------- | ---------------------------------------------------------- |
| `add set`     | Crea un conjunto.                                          |
| `add element` | Agrega elementos al conjunto (por ejemplo, IPs o puertos). |
| `list set`    | Muestra los elementos dentro de un conjunto.               |
| `del element` | Elimina elementos de un conjunto.                          |

### 7. **Ejemplos Completos**

#### 7.1 **Firewall Básico**
```bash
# Crear la tabla de filtrado
sudo nft add table inet filter

# Crear la cadena de entrada
sudo nft add chain inet filter input { type filter hook input priority 0 \; }

# Aceptar tráfico desde una IP específica
sudo nft add rule inet filter input ip saddr 192.168.1.100 accept

# Rechazar todo el tráfico no autorizado
sudo nft add rule inet filter input drop
```

#### 7.2 **Redirección de Puertos**
```bash
# Crear la tabla NAT
sudo nft add table ip nat

# Crear la cadena de prerouting para la redirección de puertos
sudo nft add chain ip nat prerouting { type nat hook prerouting priority 0 \; }

# Redirigir tráfico HTTP (puerto 80) a un servidor local en el puerto 8080
sudo nft add rule ip nat prerouting tcp dport 80 redirect to :8080
```

#### 7.3 **Masquerading (NAT de Salida)**
```bash
# Crear la tabla NAT
sudo nft add table ip nat

# Crear la cadena de postrouting para NAT de salida
sudo nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }

# Hacer masquerading en el tráfico saliente a través de la interfaz eth0
sudo nft add rule ip nat postrouting oif "eth0" masquerade
```
