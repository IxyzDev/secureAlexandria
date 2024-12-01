# Tabla Completa de Argumentos de iptables

## Comandos Principales

| Argumento | Significado                                              |
| --------- | -------------------------------------------------------- |
| -A        | Añade una regla a la cadena especificada                 |
| -P        | Establece la política por defecto para la cadena         |
| -j        | Especifica el objetivo (acción) de la regla              |
| -D        | Elimina una regla de la cadena especificada              |
| -I        | Inserta una regla en la cadena especificada              |
| -R        | Reemplaza una regla en la cadena especificada            |
| -L        | Lista las reglas en una cadena                           |
| -F        | Vacía todas las reglas en una cadena                     |
| -Z        | Resetea los contadores de bytes y paquetes en las reglas |

## Cadenas (Chains)

| Cadena      | Significado                                                                                                                         |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| INPUT       | Paquetes que llegan a la máquina                                                                                                    |
| OUTPUT      | Paquetes que salen de la máquina, antes de que sean enrutados.                                                                      |
| FORWARD     | Paquetes que pasan a través de la máquina                                                                                           |
| PREROUTING  | Modifica paquetes antes de enrutarlos (se usa con tabla `nat`)                                                                      |
| POSTROUTING | Modifica los paquetes después de que hayan sido enrutados, pero antes de que salgan de la interfaz de red. (se usa con tabla `nat`) |
| DOCKER      | Cadena utilizada por Docker para gestionar reglas específicas                                                                       |

## Direcciones IP

| Argumento | Significado                           |
| --------- | ------------------------------------- |
| -s        | Dirección IP de origen (source)       |
| --src     | Sinónimo de `-s`                      |
| -d        | Dirección IP de destino (destination) |
| --dst     | Sinónimo de `-d`                      |

## Protocolos

| Argumento  | Significado                                    |
| ---------- | ---------------------------------------------- |
| -p         | Especifica el protocolo (TCP, UDP, ICMP, etc.) |
| --protocol | Sinónimo de `-p`                               |

## Puertos

| Argumento | Significado                     |
| --------- | ------------------------------- |
| --dport   | Especifica el puerto de destino |
| --sport   | Especifica el puerto de origen  |

## Interfaz de Red

| Argumento | Significado                                                                                                                                                                  |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -i        | Especifica la interfaz de red de entrada para los paquetes. Se utiliza en las reglas que afectan a los paquetes que están entrando al sistema desde una interfaz específica. |
| -o        | Especifica la interfaz de red de salida para los paquetes. Se utiliza en las reglas que afectan a los paquetes que están saliendo del sistema hacia una interfaz específica. |

## Objetivos (Targets)

| Objetivo   | Significado                                    |
| ---------- | ---------------------------------------------- |
| ACCEPT     | Acepta el paquete                              |
| DROP       | Descarta el paquete                            |
| MASQUERADE | Enmascara la dirección IP (utilizado para NAT) |
| DNAT       | Traducción de dirección de destino             |
| SNAT       | Traducción de dirección de origen              |
| REJECT     | Rechaza el paquete y envía una respuesta ICMP  |
| LOG        | Registra el paquete en los logs del sistema    |

## Opciones Avanzadas

| Argumento        | Significado                                                                                    |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| --icmp-type      | Especifica el tipo de mensaje ICMP                                                             |
| -m               | Carga un módulo extendido (por ejemplo, `mac`, `state`)                                        |
| --mac-source     | Especifica la dirección MAC de origen                                                          |
| --state          | Especifica el estado del paquete (NEW, ESTABLISHED, RELATED)                                   |
| --limit          | Establece un límite en la tasa de coincidencia de reglas                                       |
| --limit-burst    | Especifica el número de paquetes permitidos en un burst                                        |
| -f               | Coincide con paquetes fragmentados                                                             |
| -t               | Especifica la tabla a la que se aplica la regla (`filter`, `nat`, `mangle`, `raw`, `security`) |
| --to-port        | Especifica el puerto al que se redirigirá el tráfico (para DNAT y REDIRECT)                    |
| --to-destination | Especifica la dirección IP de destino para DNAT                                                |
| --to-source      | Especifica la dirección IP de origen para SNAT                                                 |

## Ejemplos de Uso

| Ejemplo                                      | Descripción                                                                           |
| -------------------------------------------- | ------------------------------------------------------------------------------------- |
| `-A INPUT`                                   | Añadir una regla a la cadena INPUT                                                    |
| `-p tcp`                                     | Especificar que la regla aplica al protocolo TCP                                      |
| `-s 192.168.0.1`                             | Especificar que la regla aplica a paquetes desde 192.168.0.1                          |
| `-d 10.0.0.1`                                | Especificar que la regla aplica a paquetes con destino 10.0.0.1                       |
| `-j ACCEPT`                                  | Aceptar el paquete si coincide con la regla                                           |
| `-i eth0`                                    | Especificar que la regla aplica a paquetes que entran por la interfaz eth0            |
| `-o eth1`                                    | Especificar que la regla aplica a paquetes que salen por la interfaz eth1             |
| `-m state --state NEW`                       | Aplicar la regla solo a paquetes en estado NEW                                        |
| `-m limit --limit 5/minute --limit-burst 10` | Limitar la coincidencia de la regla a 5 veces por minuto, con un burst de 10 paquetes |
