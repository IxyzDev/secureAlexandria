### 1. Comandos Principales
Estos comandos son fundamentales para manipular las reglas en `iptables` y establecer la política básica de seguridad.

#### Añadir, Insertar, Eliminar y Reemplazar Reglas
| Estructura del Comando                | Ejemplo                                        |
| ------------------------------------- | ---------------------------------------------- |
| `iptables -A [CHAIN] [options]`       | `iptables -A INPUT -s 192.168.0.1 -j ACCEPT`   |
| `iptables -I [CHAIN] [num] [options]` | `iptables -I INPUT 1 -s 192.168.0.1 -j ACCEPT` |
| `iptables -D [CHAIN] [options]`       | `iptables -D INPUT -s 192.168.0.1 -j ACCEPT`   |
| `iptables -R [CHAIN] [num] [options]` | `iptables -R INPUT 1 -s 192.168.0.1 -j ACCEPT` |

#### Política por Defecto
| Estructura del Comando         | Ejemplo                  |
| ------------------------------ | ------------------------ |
| `iptables -P [CHAIN] [POLICY]` | `iptables -P INPUT DROP` |

#### Listar y Limpiar Reglas
| Estructura del Comando | Ejemplo             |
| ---------------------- | ------------------- |
| `iptables -L [CHAIN]`  | `iptables -L INPUT` |
| `iptables -F [CHAIN]`  | `iptables -F INPUT` |
| `iptables -Z [CHAIN]`  | `iptables -Z INPUT` |


### 2. Especificación de Cadena y Tabla
Definir las cadenas y tablas es crucial para organizar y aplicar correctamente las reglas de firewall.

### Cadenas

| Estructura del Comando          | Ejemplo                                      |
| ------------------------------- | -------------------------------------------- |
| `iptables -A [CHAIN] [options]` | `iptables -A INPUT -s 192.168.0.1 -j ACCEPT` |


#### Tablas
| Estructura del Comando                     | Ejemplo                                                                                   |
| ------------------------------------------ | ----------------------------------------------------------------------------------------- |
| `iptables -t [TABLE] -A [CHAIN] [options]` | `iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.0.2:80` |


### 3. Especificación de Paquetes
Estos comandos permiten definir las características de los paquetes que las reglas deben filtrar.

#### Direcciones IP
| Estructura del Comando                  | Ejemplo                                      |
| --------------------------------------- | -------------------------------------------- |
| `iptables -A [CHAIN] -s [IP] [options]` | `iptables -A INPUT -s 192.168.0.1 -j ACCEPT` |
| `iptables -A [CHAIN] -d [IP] [options]` | `iptables -A OUTPUT -d 10.0.0.1 -j ACCEPT`   |

#### Protocolos
| Estructura del Comando                        | Ejemplo                              |
| --------------------------------------------- | ------------------------------------ |
| `iptables -A [CHAIN] -p [protocol] [options]` | `iptables -A INPUT -p tcp -j ACCEPT` |


#### Puertos
| Estructura del Comando                                         | Ejemplo                                         |
| -------------------------------------------------------------- | ----------------------------------------------- |
| `iptables -A [CHAIN] -p [protocol] --dport [port] -j [target]` | `iptables -A INPUT -p tcp --dport 80 -j ACCEPT` |
| `ipt```                                                        |

#### Interfaz de Red
| Estructura del Comando                         | Ejemplo                                |
| ---------------------------------------------- | -------------------------------------- |
| `iptables -A [CHAIN] -i [interface] [options]` | `iptables -A INPUT -i eth0 -j ACCEPT`  |
| `iptables -A [CHAIN] -o [interface] [options]` | `iptables -A OUTPUT -o eth1 -j ACCEPT` |

### 4. Objetivos (Targets)
Definir las acciones que se deben tomar cuando un paquete coincide con una regla es crucial para la funcionalidad del firewall.

#### Acciones Básicas
| Estructura del Comando                    | Ejemplo                                      |
| ----------------------------------------- | -------------------------------------------- |
| `iptables -A [CHAIN] [options] -j ACCEPT` | `iptables -A INPUT -s 192.168.0.1 -j ACCEPT` |
| `iptables -A [CHAIN] [options] -j DROP`   | `iptables -A INPUT -s 192.168.0.1 -j DROP`   |
| `iptables -A [CHAIN] [options] -j REJECT` | `iptables -A INPUT -s 192.168.0.1 -j REJECT` |
| `iptables -A [CHAIN] [options] -j LOG`    | `iptables -A INPUT -s 192.168.0.1 -j LOG`    |


#### NAT y Traducción de Direcciones
| Estructura del Comando                                               | Ejemplo                                                                                   |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `iptables -t nat -A [CHAIN] [options] -j MASQUERADE`                 | `iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE`                  |
| `iptables -t nat -A [CHAIN] [options] -j DNAT --to-destination [IP]` | `iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.0.2:80` |
| `iptables -t nat -A [CHAIN] [options] -j SNAT --to-source [IP]`      | `iptables -t nat -A POSTROUTING -s 10.0.0.0/8 -j SNAT --to-source 203.0.113.1`            |

### 5. Opciones Avanzadas
Estas opciones permiten configuraciones más específicas y detalladas del firewall.

#### Estado de Conexión
| Estructura del Comando                                     | Ejemplo                                            |
| ---------------------------------------------------------- | -------------------------------------------------- |
| `iptables -A [CHAIN] -m state --state [state] -j [target]` | `iptables -A INPUT -m state --state NEW -j ACCEPT` |

#### Límites de Tasa
| Estructura del Comando                                                        | Ejemplo                                                                       |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `iptables -A [CHAIN] -m limit --limit [rate] --limit-burst [num] -j [target]` | `iptables -A INPUT -p icmp -m limit --limit 5/min --limit-burst 10 -j ACCEPT` |


#### Fragmentación
| Estructura del Comando               | Ejemplo                        |
| ------------------------------------ | ------------------------------ |
| `iptables -A [CHAIN] -f -j [target]` | `iptables -A INPUT -f -j DROP` |

#### Direcciones MAC

| Estructura del Comando                                      | Ejemplo                                                             |
| ----------------------------------------------------------- | ------------------------------------------------------------------- |
| `iptables -A [CHAIN] -m mac --mac-source [MAC] -j [target]` | `iptables -A INPUT -m mac --mac-source 00:11:22:33:44:55 -j ACCEPT` |
