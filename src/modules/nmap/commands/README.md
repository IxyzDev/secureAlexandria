## Descubrimiento de hosts
Utilizados para identificar hosts en una red.

| Flag | Descripción                            |
| ---- | -------------------------------------- |
| -sP  | Sondeo ping (sin escaneo de puertos)   |
| -PS  | TCP SYN ping                           |
| -PA  | TCP ACK ping                           |
| -PU  | UDP ping                               |
| -PY  | SCTP Init ping                         |
| -PE  | ICMP echo ping                         |
| -PP  | ICMP Timestamp ping                    |
| -PM  | ICMP address mask ping                 |
| -PO  | IP protocol ping                       |
| -PR  | ARP ping                               |
| -P0  | No realiza ping                        |
| -sL  | Sondeo de lista (lista hosts sin ping) |

### Técnicas de Rastreo
Para trazar la ruta de los paquetes a un host.

| Flag        | Descripción        |
| ----------- | ------------------ |
| -traceroute | Hace un traceroute |

### Escaneo de puertos TCP
Detecta estados de puertos TCP usando diferentes técnicas.

| Flag | Descripción                                                                                                                    |
| ---- | ------------------------------------------------------------------------------------------------------------------------------ |
| -sT  | Inicia conexiones completas. Muy fácil de detectar.                                                                            |
| -sS  | Envía un TCP SYN y espera la respuesta. Si el puerto está abierto se recibe un SYN-ACK, de lo contrario recibe un RST.         |
| -sF  | Envía un paquete vacío con FIN. Si el puerto está cerrado, se espera un RST; si está abierto, no hay respuesta.                |
| -sX  | Ataque XMAS. Activa los flags FIN, URG y PUSH. Si el puerto está cerrado, se espera un RST; si está abierto, no hay respuesta. |
| -sN  | No activa ningún flag de la cabecera TCP, se usa para análisis sigiloso.                                                       |
| -sA  | Envía un TCP ACK y se espera un RST. Utilizado para determinar si los puertos están filtrados.                                 |
| -Pn  | útil en redes donde los hosts no responden a los sondas de ping o donde el bloqueo de ping está habilitado                     |

#### Notas
* Cuando se dice que un puerto está "filtrado" en el contexto de la seguridad de redes, significa que el tráfico dirigido a ese puerto está siendo bloqueado o filtrado por un firewall o algún otro dispositivo de seguridad de red.
* Cuando se envían paquetes TCP con los flags FIN, URG y PUSH activados (Xmas Scan) a todos los puertos del objetivo, si un puerto está cerrado, el objetivo responderá con un paquete TCP RST (reset), indicando que el puerto está cerrado.

### Escaneo de puertos UDP
Identifica la respuesta de puertos UDP a diferentes tipos de paquetes.

| Flag | Descripción                                                                                                                   |
| ---- | ----------------------------------------------------------------------------------------------------------------------------- |
| -sU  | Se envía una cabecera UDP a cada puerto objetivo. Si se recibe error ICMP el puerto está cerrado; otro error indica filtrado. |

## Evasión y Detección Avanzada
Técnicas para evadir detección y obtener información de manera sigilosa.

| Flag           | Descripción                                                                  |
| -------------- | ---------------------------------------------------------------------------- |
| `-f`           | Fragmenta los paquetes                                                       |
| `--mtu <TAM>`  | Se especifica un tamaño MTU concreto                                         |
| `-S <ip>`      | Se falsea la dirección origen                                                |
| `-sl <zombie>` | Se utiliza como origen una máquina inactiva, haciéndola parecer el emisor    |
| `-D <señuelo>` | Se utilizan equipos señuelo para simular múltiples IPs de origen del escaneo |
| `-g <puerto>`  | Se falsea el puerto de origen                                                |

## Analisis de servicios
| Flag             | Descripción                                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------------------------- |
| `-sV`            | Detección de las versiones de los servicios                                                                   |
| `--all-ports`    | Escanea todos los puertos                                                                                     |
| `-O`             | Descubrimiento de la versión del S.O.                                                                         |
| `--osscan-guess` | Intenta adivinar S.O. por aproximación                                                                        |
| `-sR`            | Se envían ordenes de programa NULL SunRPC a todos los puertos para determinar si son puertos RPC y su versión |
| -sW              | Se utilizan barridos "-PI" y "-PT" en paralelo. Aprovecha detalles para listar puertos de manera eficaz.      |

### Descubrimiento de hosts y técnicas adicionales
Además del descubrimiento de hosts tradicional, estos flags se usan para eludir firewalls y detectar servicios ocultos o protegidos.

| Flag | Descripción                                                                                                       |
| ---- | ----------------------------------------------------------------------------------------------------------------- |
| -sP  | Utilizado para descubrimiento de Sistemas. Envía un ICMP y un TCP SYN al puerto 80.                               |
| -sI  | Se utilizan paquetes ICMP para intentar eludir firewalls y detectar hosts y servicios que no responden a TCP/UDP. |

## Formato de salida
| Flag            | Descripción                  |
| --------------- | ---------------------------- |
| `-oN <fichero>` | Redirige la salida normal    |
| `-oX <fichero>` | Salida en XML                |
| `-oS <fichero>` | Salida «script kiddie»       |
| `-oG <fichero>` | Salida «grepable»            |
| `-oA <fichero>` | Salida en todos los formatos |


## nmap scripting engine
| Flag                                     | Descripción                          |
| ---------------------------------------- | ------------------------------------ |
| `-sC`                                    | Ejecuta los scripts por defecto      |
| `--script=<ScriptName>\<ScriptCategory>` | Ejecuta uno o más scripts            |
| `--script-args=<Name1=Value1,...>`       | Usa la lista de argumentos de script |
| `--script-updatedb`                      | Actualiza la base de datos de script |

## Categorias de scripts
| Categoría | Descripción                                                                                                                                  |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| auth      | ejecuta todos sus scripts disponibles para autenticación                                                                                     |
| broadcast | Descubra hosts no incluidos en la línea de comando transmitiendo en la red local                                                             |
| brute     | Intente adivinar las contraseñas en los sistemas objetivo, para una variedad de protocolos                                                   |
| default   | Scripts que se ejecutan por defecto al poner los flags -sC o -A                                                                              |
| discovery | Intenta obtener más información sobre los hosts de destino a través de fuentes públicas de información, SNMP, servicios de directorio y más. |
| dos       | Puede causar condiciones de denegación de servicio en los hosts objetivos.                                                                   |
| exploit   | Intenta explotar los sistemas objetivos                                                                                                      |
| external  | Interactuar con sistemas de terceros no incluidos en la lista de objetivos                                                                   |
| fuzzer    | Envía entradas inesperadas a los campos de los protocolo de red.                                                                             |
| intrusive | Puede bloquear el objetivo, consumir recursos excesivos o afectar de manera maliciosa las máquinas.                                          |
| malware   | Busca signos de una infección malware en el host objetivo                                                                                    |
| safe      | Diseñado para no afectar al objetivo de manera negativa                                                                                      |
| version   | Mide la versión del software o protocolo usado por el host objetivo                                                                          |
| vul       | Busca si los sistemas objetivos, tienen alguna vulnerabilidad conocida                                                                       |


## Ver Paquetes recibidos y enviados por Nmap
```bash	
nmap -sS [IP] -p [Puerto], [Puerto], [Puerto] --packet-trace
```

