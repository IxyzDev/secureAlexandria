Para aumentar la seguridad de un servidor conectado a Internet y con servicios web, MySQL y FTP, es crucial restringir el acceso únicamente a las direcciones IP y puertos necesarios. A continuación, se presentan las reglas adicionales que deben aplicarse:

### Políticas por Defecto

#### Políticas por Defecto para INPUT, OUTPUT y FORWARD
Establecemos una política por defecto más restrictiva:
```bash
# Establecer políticas por defecto más restrictivas
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```

### FLUSH de Reglas
Limpiamos todas las reglas existentes:

```bash
iptables -F
```
* Se utiliza para eliminar todas las reglas actuales en iptables antes de aplicar nuevas reglas. Es una buena práctica para evitar conflictos o redundancias.

```bash
iptables -X
```
* Se utiliza para limpiar cualquier cadena personalizada que haya sido creada, asegurando que solo las cadenas predeterminadas (INPUT, OUTPUT, FORWARD) permanezcan.

```bash
iptables -Z
```
* Se utiliza para reiniciar los contadores de estadísticas, útil para monitorear nuevas reglas desde cero.

```bash
iptables -t nat -F
```
* Se utiliza para eliminar todas las reglas de NAT (Network Address Translation) antes de aplicar nuevas reglas. Esto asegura que no haya reglas NAT conflictivas o redundantes.

### Reglas Básicas
Permitimos acceso a la IP del administrador, al DBA para MySQL, al operador para FTP y abrimos el puerto web (80):
```bash
# Permitir acceso completo a la IP del administrador (ejemplo: 192.168.0.1)
iptables -A INPUT -s 192.168.0.1 -j ACCEPT

# Permitir acceso al puerto MySQL (3306) para el DBA (ejemplo: 192.168.0.2)
iptables -A INPUT -p tcp -s 192.168.0.2 --dport 3306 -j ACCEPT

# Cerramos acceso al puerto MySQL (3306) para todos los demás
iptables −A INPUT −p tcp −−dport 3306 −j DROP

# Permitir acceso al puerto FTP (21) para el operador (ejemplo: 192.168.0.3)
iptables -A INPUT -p tcp -s 192.168.0.3 --dport 20,21 -j ACCEPT
"""
Necesidad del Servicio FTP:
Si el operador necesita transferir archivos hacia o desde el servidor, se requiere permitir el acceso al puerto 21 donde el servidor FTP escucha las conexiones de control.
"""

# Permitir acceso al puerto web (80) para todos
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Permitir acceso al puerto HTTPS (443) para todos
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Ceramos el resto de los puertos
iptables -A INPUT -p tcp --dport 1:1024 -j DROP
iptables -A INPUT -p udp --dport 1:1024 -j DROP
```



### Reglas Adicionales para Seguridad

1. **Permitir tráfico de loopback (localhost):**
```bash
iptables -A INPUT -i lo -j ACCEPT
```
* Esta regla permite todo el tráfico entrante desde la interfaz de loopback (localhost). La interfaz de loopback es una interfaz de red virtual utilizada por el sistema operativo para comunicarse consigo mismo. La dirección IP asociada comúnmente con esta interfaz es 127.0.0.1.

2. **Permitir conexiones establecidas y relacionadas:**
```bash
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```
* Permitir conexiones establecidas y relacionadas es una regla fundamental en la configuración de un firewall porque ayuda a asegurar que el tráfico no solicitado no pueda acceder a tu red interna. Esta regla permite que las respuestas a las solicitudes salientes y las conexiones establecidas se procesen sin problemas.

3. **Permitir ICMP (ping) de forma limitada:**
```bash
iptables -A INPUT -p icmp -m limit --limit 1/s --limit-burst 5 -j ACCEPT
```
* Esta regla permite los paquetes ICMP, que son utilizados para mensajes de control de red como ping, pero los limita para prevenir abusos como ataques de denegación de servicio (DoS).

4. **Limitar el acceso SSH (puerto 22) solo desde direcciones IP específicas (ejemplo: 192.168.0.4):**
```bash
iptables -A INPUT -p tcp -s 192.168.0.4 --dport 22 -j ACCEPT
```
* Esta regla permite el acceso SSH (puerto 22) solo desde la dirección IP específica 192.168.0.4. Esto significa que solo el dispositivo con la IP 192.168.0.4 podrá iniciar una conexión SSH con el servidor.

5. **Registrar y rechazar todo el tráfico no permitido:**
```bash
iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "
iptables -A INPUT -j REJECT
```
* Estas reglas primero registran los paquetes que no coinciden con ninguna regla permitida y luego los rechazan. Esto es útil para el diagnóstico y la seguridad, ya que proporciona un registro de los intentos de acceso no autorizados y asegura que estos paquetes sean rechazados de manera activa.

6. **Bloquear puertos no utilizados:**
```bash
# Bloquear todos los demás puertos no utilizados explícitamente
iptables -A INPUT -p tcp --dport 1:19 -j DROP
iptables -A INPUT -p tcp --dport 23:79 -j DROP
iptables -A INPUT -p tcp --dport 81:442 -j DROP
iptables -A INPUT -p tcp --dport 444:65535 -j DROP
```
* Estas reglas bloquean todos los paquetes entrantes a través del protocolo TCP que se dirigen a puertos no utilizados explícitamente dentro de los rangos especificados. Esto mejora la seguridad al reducir la superficie de ataque de la máquina, asegurando que solo los puertos necesarios estén abiertos y accesibles.


### Ejemplo Completo
```bash
echo -n "Aplicando Reglas de Firewall..."

# FLUSH de reglas
iptables -F
iptables -X
iptables -Z
iptables -t nat -F

# Políticas por defecto
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

# Permitir tráfico loopback
iptables -A INPUT -i lo -j ACCEPT

# Permitir conexiones establecidas y relacionadas
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Permitir ICMP (ping) de forma limitada
iptables -A INPUT -p icmp -m limit --limit 1/s --limit-burst 5 -j ACCEPT

# Permitir acceso completo a la IP del administrador
iptables -A INPUT -s 192.168.0.1 -j ACCEPT

# Permitir acceso al puerto MySQL (3306) para el DBA
iptables -A INPUT -p tcp -s 192.168.0.2 --dport 3306 -j ACCEPT

# Permitir acceso al puerto FTP (21) para el operador
iptables -A INPUT -p tcp -s 192.168.0.3 --dport 21 -j ACCEPT

# Permitir acceso al puerto web (80) para todos
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Permitir acceso al puerto HTTPS (443) para todos
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Limitar el acceso SSH (puerto 22) solo desde direcciones IP específicas
iptables -A INPUT -p tcp -s 192.168.0.4 --dport 22 -j ACCEPT

# Registrar y rechazar todo el tráfico no permitido
iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "
iptables -A INPUT -j REJECT

# Bloquear puertos no utilizados
iptables -A INPUT -p tcp --dport 1:19 -j DROP
iptables -A INPUT -p tcp --dport 23:79 -j DROP
iptables -A INPUT -p tcp --dport 81:442 -j DROP
iptables -A INPUT -p tcp --dport 444:65535 -j DROP

echo "Reglas de Firewall aplicadas."
```

Estas reglas proporcionan una configuración de firewall básica pero segura, protegiendo los servicios web, MySQL y FTP, y restringiendo el acceso solo a las direcciones IP y puertos necesarios.

## Configuración de Firewall para la Red LAN

El esquema de red presentado utiliza un firewall para proteger la red LAN y controlar el acceso a Internet. A continuación, se describen las configuraciones necesarias y los problemas potenciales de este esquema.

### Configuración Necesaria

#### 1. Acceso desde la Red Local

```bash
# Permitir acceso desde la red local
iptables -A INPUT -s 192.168.10.0/24 -i eth1 -j ACCEPT
```

#### 2. Redirecciones
   
```bash
# Redirigir tráfico del puerto 80 al servidor interno
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.10.12:80

# Redirigir tráfico del puerto 3389 desde una IP específica al servidor interno
iptables -t nat -A PREROUTING -s 221.23.124.181 -i eth0 -p tcp --dport 3389 -j DNAT --to-destination 192.168.10.12:3389

```
#### 3. Abrir Puertos de Correo

```bash
# Abrir puerto 25 (SMTP)
iptables -A INPUT -s 0.0.0.0/0 -p tcp --dport 25 -j ACCEPT

# Abrir puerto 110 (POP3)
iptables -A INPUT -s 0.0.0.0/0 -p tcp --dport 110 -j ACCEPT

# Abrir puerto 1723 (PPTP) para una IP específica
iptables -A INPUT -s 211.45.176.24 -p tcp --dport 1723 -j ACCEPT
```

#### 4. Permitir Acceso a Servicios Web y DNS desde la LAN
```bash
# Permitir tráfico HTTP desde la LAN
iptables -A FORWARD -s 192.168.10.0/24 -i eth1 -p tcp --dport 80 -j ACCEPT

# Permitir tráfico HTTPS desde la LAN
iptables -A FORWARD -s 192.168.10.0/24 -i eth1 -p tcp --dport 443 -j ACCEPT

# Permitir consultas DNS desde la LAN
iptables -A FORWARD -s 192.168.10.0/24 -i eth1 -p tcp --dport 53 -j ACCEPT
iptables -A FORWARD -s 192.168.10.0/24 -i eth1 -p udp --dport 53 -j ACCEPT

# Denegar todo el tráfico restante desde la LAN
iptables -A FORWARD -s 192.168.10.0/24 -i eth1 -j DROP

```

#### 5. Configurar NAT y Habilitar el Bit de Forwarding
El enmascaramiento (masquerading) permite que múltiples dispositivos en una red local (LAN) compartan una única dirección IP pública, facilitando el acceso a Internet. Por ejemplo, si tienes una red interna `192.168.10.0/24` y una conexión a Internet a través de la interfaz `eth0`, la regla de enmascaramiento permite que todos los dispositivos en `192.168.10.0/24` accedan a Internet utilizando la dirección IP pública de `eth0`.

```bash
# Configuración de NAT en la cadena POSTROUTING para enmascarar tráfico saliente
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE
```
Bit de Forwarding: Una configuración en las interfaces de red que determina si el sistema debe reenviar paquetes entre diferentes interfaces de red.
Propósito: Permitir que un sistema Linux actúe como un router, encaminando tráfico de una red a otra.

#### 6. Cerramos el rango de puerto bien conocido

```bash
# Cerrar el rango de puertos bien conocidos (0-1024) para TCP
iptables -A INPUT -s 0.0.0.0/0 -p tcp --dport 1:1024 -j DROP

# Cerrar el rango de puertos bien conocidos (0-1024) para UDP
iptables -A INPUT -s 0.0.0.0/0 -p udp --dport 1:1024 -j DROP

# Cerrar el puerto 1723 para PPTP excepto para la IP específica del jefe
iptables -A INPUT -s 0.0.0.0/0 -i eth0 -p tcp --dport 1723 -j DROP
```

### Problemas Potenciales del Esquema

1. **Seguridad Insuficiente:**
   - **Telnet:** Telnet no es seguro ya que transmite los datos en texto claro, lo que lo hace vulnerable a la intercepción. Es recomendable usar SSH en lugar de Telnet.
   - **Puertos Abiertos:** Tener múltiples servicios abiertos (SMTP, POP3, PPTP) puede aumentar la superficie de ataque. Cada servicio expuesto es un posible punto de entrada para atacantes.

2. **Rendimiento del Firewall:**
   - **Carga de Procesamiento:** El firewall tendrá que manejar el NAT y el filtrado de paquetes, lo que puede ser una carga considerable dependiendo del volumen de tráfico.

3. **Complejidad de la Configuración:**
   - **Doble NAT:** La configuración de doble NAT puede complicar el diagnóstico de problemas de conectividad y puede introducir latencia adicional.

4. **Gestión Centralizada:**
   - **Punto Único de Falla:** El firewall se convierte en un punto único de falla. Si el firewall falla, toda la conectividad de la red LAN a Internet se verá interrumpida.

5. **Acceso Interno:**
   - **Políticas de Seguridad Interna:** No hay indicación de políticas de seguridad interna para proteger la LAN contra amenazas internas. Es crucial implementar reglas que limiten el tráfico dentro de la LAN misma.

6. **Logs y Monitoreo:**
   - **Falta de Monitoreo:** No hay mención de registro y monitoreo del tráfico, lo cual es crucial para detectar y responder a posibles incidentes de seguridad.

### Soluciones y Recomendaciones

1. **Seguridad Mejorada:**
   - **Usar SSH en lugar de Telnet:** Configurar el acceso remoto usando SSH para una mayor seguridad.
   - **Revisar Puertos Abiertos:** Solo abrir los puertos necesarios y utilizar herramientas de auditoría de seguridad para revisar regularmente las configuraciones.

2. **Optimización del Firewall:**
   - **Hardware Adecuado:** Asegurarse de que el hardware del firewall pueda manejar el volumen de tráfico esperado sin degradar el rendimiento.

3. **Simplificación de NAT:**
   - **Simplificar la Configuración:** Evaluar si el doble NAT es realmente necesario o si se puede simplificar la topología de red.

4. **Monitoreo y Respuesta:**
   - **Implementar Logging:** Habilitar el registro de tráfico y utilizar herramientas de monitoreo para identificar y responder a incidentes de seguridad.

5. **Seguridad Interna:**
   - **Políticas de Segmentación:** Implementar políticas de segmentación de la red para limitar el tráfico entre dispositivos en la LAN.

Con estas medidas, se puede mejorar significativamente la seguridad y la eficiencia del esquema de red presentado.

-----------------------------------------------
**NAT entre el router y el firewall:**
```bash
# Enmascarar el tráfico saliente desde la red interna hacia el router
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

**NAT entre el firewall y la red LAN:**
```bash
# Enmascarar el tráfico saliente desde la LAN hacia el firewall
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth1 -j MASQUERADE
```

Para mejorar la seguridad del esquema y proteger la red interna utilizando una DMZ (Zona Desmilitarizada), se deben establecer reglas específicas en el firewall. Aquí está la configuración recomendada basada en los requisitos que has mencionado.

### Configuración del Firewall para DMZ

#### 1. Establecer las políticas por defecto

```bash
# Establecer políticas por defecto más restrictivas
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
iptables -t nat -P PREROUTING ACCEPT
iptables -t nat -P POSTROUTING ACCEPT
```

#### 2. FLUSH de reglas
```bash
# Limpiar reglas existentes
iptables -F
iptables -X
iptables -Z
iptables -t nat -F
```

#### 3. Permitir tráfico esencial

**Permitir tráfico de loopback (localhost):**
```bash
iptables -A INPUT -i lo -j ACCEPT
```

**Permitir conexiones establecidas y relacionadas:**
```bash
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

#### 4. Acceso de la red local a Internet
```bash
# Permitir acceso de la red local a Internet
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.10.0/24 -j ACCEPT

# Permitr el acceso local a firewall
iptables −A INPUT −s 192.168.10.0/24 −i eth1 −j ACCEPT

# Enmascarar el tráfico saliente desde la LAN
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE

```

#### 5. Acceso público a los puertos HTTP (80) y HTTPS (443) del servidor en la DMZ
```bash
# Permitir tráfico HTTP y HTTPS a la DMZ
iptables -A FORWARD -p tcp -d 192.168.3.1 --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp -d 192.168.3.1 --dport 443 -j ACCEPT

# Redirigir tráfico HTTP y HTTPS a la DMZ
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.3.1:80
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 192.168.3.1:443
```

#### 6. Acceso del servidor en la DMZ a la base de datos en la LAN
```bash
# Permitir acceso a la base de datos desde la DMZ
iptables −A FORWARD −s 192.168.3.2 −d 192.168.10.5 −p tcp −−dport 5432 −j ACCEPT
iptables −A FORWARD −s 192.168.10.5 −d 192.168.3.2 −p tcp −−sport 5432 −j ACCEPT

# permitimos abrir el Terminal server de la DMZ desde la LAN
iptables −A FORWARD −s 192.168.10.0/24 −d 192.168.3.2 −p tcp −−sport 1024:65535 −−dport 3389 −j ACCEPT

# Se debe hacer en uno y otro sentido …
iptables −A FORWARD −s 192.168.3.2 −d 192.168.10.0/24 −p tcp −−sport 3389 −−dport 1024:65535 −j ACCEPT
```

#### 7. Bloquear el resto de acceso de la DMZ hacia la LAN
```bash
# Bloquear todo el tráfico de la DMZ hacia la LAN excepto lo permitido
iptables -A FORWARD -s 192.168.3.0/24 -d 192.168.10.0/24 -j DROP

# Cerramos el acceso de la DMZ al propio firewall
iptables −A INPUT −s 192.168.3.0/24 −i eth2 −j DROP

# Cerramos el rango de puerto bien conocido
iptables −A INPUT −s 0.0.0.0/0 −p tcp −dport 1:1024 −j DROP
iptables −A INPUT −s 0.0.0.0/0 −p udp −dport 1:1024 −j DROP
# Cerramos un puerto de gestión: webmin
iptables −A INPUT −s 0.0.0.0/0 −p tcp −dport 10000 −j DROP
```

### Problemas del Esquema Anterior (Sin DMZ)

Si el servidor web en la red local es hackeado, el atacante podría tener acceso a la red interna, comprometiendo otros servicios y dispositivos. Además, los ataques desde el servidor comprometido podrían propagarse a otros sistemas dentro de la LAN.

### Soluciones con DMZ

Usar una DMZ proporciona una capa adicional de seguridad al separar los servicios accesibles desde Internet (como el servidor web) de la red interna. Esto reduce el riesgo de que un atacante comprometa la red interna si el servidor en la DMZ es hackeado.

### Resumen de Reglas Necesarias

1. **Permitir acceso de la red local a Internet.**
2. **Permitir acceso público a los puertos HTTP (80) y HTTPS (443) del servidor en la DMZ.**
3. **Permitir acceso del servidor en la DMZ a la base de datos en la LAN.**
4. **Bloquear todo el tráfico de la DMZ hacia la LAN, excepto el tráfico permitido.**

### Ejemplo Completo

```bash
echo -n "Aplicando Reglas de Firewall..."

# FLUSH de reglas
iptables -F
iptables -X
iptables -Z
iptables -t nat -F

# Políticas por defecto
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

# Permitir tráfico loopback
iptables -A INPUT -i lo -j ACCEPT

# Permitir conexiones establecidas y relacionadas
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# Permitir acceso de la red local a Internet
iptables -A FORWARD -i eth1 -o eth0 -s 192.168.10.0/24 -j ACCEPT

# Enmascarar el tráfico saliente desde la LAN
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE

# Permitir tráfico HTTP y HTTPS a la DMZ
iptables -A FORWARD -p tcp -d 192.168.3.1 --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp -d 192.168.3.1 --dport 443 -j ACCEPT

# Redirigir tráfico HTTP y HTTPS a la DMZ
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.3.1:80
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 192.168.3.1:443

# Permitir acceso a la base de datos desde la DMZ
iptables -A FORWARD -p tcp -s 192.168.3.1 -d 192.168.10.2 --dport 3306 -j ACCEPT

# Bloquear todo el tráfico de la DMZ hacia la LAN excepto lo permitido
iptables -A FORWARD -s 192.168.3.0/24 -d 192.168.10.0/24 -j DROP

echo "Reglas de Firewall aplicadas."
```

Con estas reglas, el firewall configurará una DMZ segura que permite el acceso controlado desde Internet al servidor web, permite que el servidor en la DMZ acceda a la base de datos en la LAN y bloquea cualquier otro acceso de la DMZ a la LAN, mejorando así la seguridad del esquema de red.

### Configuración de Firewall con DMZ para Mejorar la Seguridad de la Red

#### Contexto
La configuración de un firewall para proteger la red LAN y controlar el acceso a Internet se torna insegura si no se implementan medidas adicionales como una DMZ (Zona Desmilitarizada). La DMZ permite aislar los servicios públicos del resto de la red, reduciendo el riesgo de comprometer la red interna en caso de que un servidor público sea hackeado.

#### Objetivos de la Configuración
1. Acceso de la red local a Internet.
2. Acceso público al puerto TCP/80 y TCP/443 del servidor de la DMZ.
3. Acceso del servidor de la DMZ a una base de datos en la LAN.
4. Bloquear el resto del acceso de la DMZ hacia la LAN.

### Reglas Necesarias para Filtrar el Tráfico entre la DMZ y la LAN

#### FLUSH de Reglas Anteriores
```bash
iptables -F
iptables -X
iptables -Z
iptables -t nat -F
```

#### Establecer Políticas por Defecto
```bash
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -t nat -P PREROUTING ACCEPT
iptables -t nat -P POSTROUTING ACCEPT
```

#### Redirecciones y NAT
```bash
# Redirigir tráfico del puerto 80 a la máquina interna
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 192.168.3.2:80

# Redirigir tráfico del puerto 443 a la máquina interna
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to 192.168.3.2:443

# Permitir conexiones locales al firewall
iptables -A INPUT -i lo -j ACCEPT

# Permitir acceso al firewall desde la red local
iptables -A INPUT -s 192.168.10.0/24 -i eth1 -j ACCEPT

# Realizar el enmascaramiento de la red local y la DMZ
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -o eth0 -j MASQUERADE

# Habilitar el bit de forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### Permitir el Paso de la DMZ a la Base de Datos en la LAN
```bash
iptables -A FORWARD -s 192.168.3.2 -d 192.168.10.5 -p tcp --dport 5432 -j ACCEPT
iptables -A FORWARD -s 192.168.10.5 -d 192.168.3.2 -p tcp --sport 5432 -j ACCEPT
```

#### Permitir Acceso al Terminal Server de la DMZ desde la LAN
```bash
iptables -A FORWARD -s 192.168.10.0/24 -d 192.168.3.2 -p tcp --sport 1024:65535 --dport 3389 -j ACCEPT
iptables -A FORWARD -s 192.168.3.2 -d 192.168.10.0/24 -p tcp --sport 3389 --dport 1024:65535 -j ACCEPT
```

#### Bloquear Acceso de la DMZ a la LAN
```bash
iptables -A FORWARD -s 192.168.3.0/24 -d 192.168.10.0/24 -j DROP
```

#### Bloquear Acceso de la DMZ al Propio Firewall
```bash
iptables -A INPUT -s 192.168.3.0/24 -i eth2 -j DROP
```

#### Bloquear Accesos Indeseados desde el Exterior
```bash
# Cerrar el rango de puertos bien conocidos
iptables -A INPUT -s 0.0.0.0/0 -p tcp --dport 1:1024 -j DROP
iptables -A INPUT -s 0.0.0.0/0 -p udp --dport 1:1024 -j DROP

# Cerrar el puerto de gestión (Webmin)
iptables -A INPUT -s 0.0.0.0/0 -p tcp --dport 10000 -j DROP
```

### Problema con IPs Públicas en la DMZ

Si las máquinas de la DMZ tienen una IP pública, se debe tener cuidado de no permitir el FORWARD por defecto. En este caso, no es necesario realizar redirecciones de puerto, sino que basta con enrutar los paquetes para que lleguen a la DMZ. 

### Configuración Sugerida sin Redirecciones para IPs Públicas en la DMZ
```bash
# Permitir el tráfico de la red local a Internet
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Permitir acceso público al puerto TCP/80 y TCP/443 del servidor de la DMZ
iptables -A FORWARD -i eth0 -p tcp --dport 80 -d 192.168.3.2 -j ACCEPT
iptables -A FORWARD -i eth0 -p tcp --dport 443 -d 192.168.3.2 -j ACCEPT

# Permitir acceso del servidor de la DMZ a la base de datos en la LAN
iptables -A FORWARD -s 192.168.3.2 -d 192.168.10.5 -p tcp --dport 5432 -j ACCEPT
iptables -A FORWARD -d 192.168.3.2 -s 192.168.10.5 -p tcp --sport 5432 -j ACCEPT

# Bloquear el resto del acceso de la DMZ hacia la LAN
iptables -A FORWARD -s 192.168.3.0/24 -d 192.168.10.0/24 -j DROP
```

### Resumen

Este conjunto de reglas establece un firewall seguro que utiliza una DMZ para proteger la red interna. Permite el acceso necesario a servicios públicos, mientras restringe el acceso no autorizado y asegura que el tráfico se maneje de manera segura.

A continuación, se presentan las reglas de `iptables` para cada uno de los puntos mencionados:

### 1. Añadir una regla a la cadena INPUT para aceptar todos los paquetes que se originan desde la dirección 192.168.106.200.
```bash
iptables -A INPUT -s 192.168.106.200 -j ACCEPT
```

### 2. Eliminar todos los paquetes que entren.
```bash
iptables -P INPUT DROP
```

### 3. Permitir la salida de paquetes.
```bash
iptables -P OUTPUT ACCEPT
```

### 4. Añadir una regla a la cadena INPUT para rechazar todos los paquetes que se originan desde la dirección 192.168.106.200.
```bash
iptables -A INPUT -s 192.168.106.200 -j REJECT
```

### 5. Añadir una regla a la cadena INPUT para rechazar todos los paquetes que se originan desde la dirección de red 192.168.0.0.
```bash
iptables -A INPUT -s 192.168.0.0/16 -j REJECT
```

### 6. Permitir el acceso al servidor web (puerto TCP 80).
```bash
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

### 7. Permitir el acceso a nuestro servidor FTP (puerto TCP 20 y 21).
```bash
iptables -A INPUT -p tcp --dport 20 -j ACCEPT
iptables -A INPUT -p tcp --dport 21 -j ACCEPT
```

### 8. Permitimos a la máquina con IP 192.168.106.200 conectarse por medio de SSH.
```bash
iptables -A INPUT -p tcp -s 192.168.106.200 --dport 22 -j ACCEPT
```

### 9. Rechazamos a la máquina con IP 192.168.106.200 conectarse por medio de Telnet.
```bash
iptables -A INPUT -p tcp -s 192.168.106.200 --dport 23 -j REJECT
```

### 10. Rechazamos todo el tráfico que ingrese a nuestra red LAN 192.168.0.0/24 desde una red remota, como Internet, a través de la interfaz eth0.
```bash
iptables -A INPUT -i eth0 -s 0.0.0.0/0 -d 192.168.0.0/24 -j REJECT
```

### 11. Cerramos el rango de puertos bien conocidos (1-1023) desde cualquier origen.
```bash
iptables -A INPUT -p tcp --dport 1:1023 -j REJECT
iptables -A INPUT -p udp --dport 1:1023 -j REJECT
```

### 12. Aceptamos que vayan de nuestra red 192.168.0.0/24 a un servidor web (puerto 80).
```bash
iptables -A FORWARD -s 192.168.0.0/24 -p tcp --dport 80 -j ACCEPT
```

### 13. Aceptamos que nuestra LAN 192.168.0.0/24 vaya a puertos HTTPS.
```bash
iptables -A FORWARD -s 192.168.0.0/24 -p tcp --dport 443 -j ACCEPT
```

### 14. Aceptamos que los equipos de nuestra red LAN 192.168.0.0/24 consulten los DNS, y denegamos todo el resto a nuestra red.
```bash
iptables -A FORWARD -s 192.168.0.0/24 -p tcp --dport 53 -j ACCEPT
iptables -A FORWARD -s 192.168.0.0/24 -p udp --dport 53 -j ACCEPT
iptables -A FORWARD -d 192.168.0.0/24 -j REJECT
```

### 15. Permitimos enviar y recibir e-mail a todos.
```bash
iptables -A INPUT -p tcp --dport 25 -j ACCEPT
iptables -A INPUT -p tcp --dport 587 -j ACCEPT
iptables -A INPUT -p tcp --dport 993 -j ACCEPT
iptables -A INPUT -p tcp --dport 995 -j ACCEPT
```

### 16. Cerramos el acceso de una red definida 192.168.3.0/24 a nuestra red LAN 192.168.2.0/24.
```bash
iptables -A FORWARD -s 192.168.3.0/24 -d 192.168.2.0/24 -j REJECT
```

### 17. Permitimos el tráfico TCP y UDP de un equipo específico 192.168.3.5 a un servicio (puerto 5432) que ofrece un equipo específico (192.168.0.5) y su respuesta.
```bash
iptables -A FORWARD -p tcp -s 192.168.3.5 -d 192.168.0.5 --dport 5432 -j ACCEPT
iptables -A FORWARD -p udp -s 192.168.3.5 -d 192.168.0.5 --dport 5432 -j ACCEPT
iptables -A FORWARD -p tcp -s 192.168.0.5 -d 192.168.3.5 --sport 5432 -j ACCEPT
iptables -A FORWARD -p udp -s 192.168.0.5 -d 192.168.3.5 --sport 5432 -j ACCEPT
```

### 18. Permitimos el paso de paquetes cuya conexión se encuentra establecida.
```bash
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```
Aquí tienes una síntesis de los comandos de `iptables` más utilizados, explicando su estructura, qué hacen, cuándo se utilizan y en qué ejercicios se aplicaron. Esta recopilación te ayudará a prepararte para un examen académico con rigor y especificidad.

### Comandos y Argumentos de `iptables`

#### 1. `-A` (append)
**Estructura:**
```bash
iptables -A [CHAIN] [options]
```
**Descripción:**
Añade una regla al final de una cadena especificada.
**Uso:**
Se utiliza para agregar nuevas reglas a las cadenas de `iptables`.
**Ejemplos:**
1. Aceptar paquetes desde una IP específica.
   ```bash
   iptables -A INPUT -s 192.168.106.200 -j ACCEPT
   ```
2. Permitir acceso al puerto HTTP.
   ```bash
   iptables -A INPUT -p tcp --dport 80 -j ACCEPT
   ```
3. Permitir acceso al puerto FTP.
   ```bash
   iptables -A INPUT -p tcp --dport 21 -j ACCEPT
   ```

#### 2. `-P` (policy)
**Estructura:**
```bash
iptables -P [CHAIN] [POLICY]
```
**Descripción:**
Establece la política por defecto para una cadena especificada.
**Uso:**
Se utiliza para definir el comportamiento predeterminado de una cadena cuando no se cumplen otras reglas.
**Ejemplos:**
1. Eliminar todos los paquetes que entren.
   ```bash
   iptables -P INPUT DROP
   ```
2. Permitir la salida de paquetes.
   ```bash
   iptables -P OUTPUT ACCEPT
   ```

#### 3. `-s` (source)
**Estructura:**
```bash
iptables -A [CHAIN] -s [IP] [options]
```
**Descripción:**
Especifica la dirección IP de origen de los paquetes.
**Uso:**
Se utiliza para aplicar reglas a paquetes que provienen de una dirección IP específica.
**Ejemplos:**
1. Rechazar paquetes desde una IP específica.
   ```bash
   iptables -A INPUT -s 192.168.106.200 -j REJECT
   ```
2. Aceptar paquetes desde una subred.
   ```bash
   iptables -A FORWARD -s 192.168.0.0/24 -p tcp --dport 80 -j ACCEPT
   ```

#### 4. `-d` (destination)
**Estructura:**
```bash
iptables -A [CHAIN] -d [IP] [options]
```
**Descripción:**
Especifica la dirección IP de destino de los paquetes.
**Uso:**
Se utiliza para aplicar reglas a paquetes que se dirigen a una dirección IP específica.
**Ejemplos:**
1. Rechazar tráfico que ingrese a una red específica.
   ```bash
   iptables -A INPUT -i eth0 -s 0.0.0.0/0 -d 192.168.0.0/24 -j REJECT
   ```

#### 5. `-p` (protocol)
**Estructura:**
```bash
iptables -A [CHAIN] -p [protocol] [options]
```
**Descripción:**
Especifica el protocolo de los paquetes (TCP, UDP, ICMP, etc.).
**Uso:**
Se utiliza para aplicar reglas a paquetes de un protocolo específico.
**Ejemplos:**
1. Permitir acceso al puerto HTTPS.
   ```bash
   iptables -A INPUT -p tcp --dport 443 -j ACCEPT
   ```
2. Permitir acceso al puerto DNS.
   ```bash
   iptables -A FORWARD -s 192.168.0.0/24 -p udp --dport 53 -j ACCEPT
   ```

#### 6. `--dport` (destination port)
**Estructura:**
```bash
iptables -A [CHAIN] -p [protocol] --dport [port] -j [target]
```
**Descripción:**
Especifica el puerto de destino de los paquetes.
**Uso:**
Se utiliza para aplicar reglas a paquetes que se dirigen a un puerto específico.
**Ejemplos:**
1. Permitir acceso al servidor web.
   ```bash
   iptables -A INPUT -p tcp --dport 80 -j ACCEPT
   ```
2. Rechazar acceso Telnet.
   ```bash
   iptables -A INPUT -p tcp -s 192.168.106.200 --dport 23 -j REJECT
   ```

#### 7. `--sport` (source port)
**Estructura:**
```bash
iptables -A [CHAIN] -p [protocol] --sport [port] -j [target]
```
**Descripción:**
Especifica el puerto de origen de los paquetes.
**Uso:**
Se utiliza para aplicar reglas a paquetes que provienen de un puerto específico.
**Ejemplos:**
1. Permitir tráfico de respuesta de un servidor.
   ```bash
   iptables -A FORWARD -p tcp -s 192.168.0.5 -d 192.168.3.5 --sport 5432 -j ACCEPT
   ```

#### 8. `-i` (input interface)
**Estructura:**
```bash
iptables -A [CHAIN] -i [interface] [options]
```
**Descripción:**
Especifica la interfaz de red de entrada.
**Uso:**
Se utiliza para aplicar reglas a paquetes que entran por una interfaz específica.
**Ejemplos:**
1. Rechazar tráfico desde una red remota a través de la interfaz eth0.
   ```bash
   iptables -A INPUT -i eth0 -s 0.0.0.0/0 -d 192.168.0.0/24 -j REJECT
   ```

#### 9. `-o` (output interface)
**Estructura:**
```bash
iptables -A [CHAIN] -o [interface] [options]
```
**Descripción:**
Especifica la interfaz de red de salida.
**Uso:**
Se utiliza para aplicar reglas a paquetes que salen por una interfaz específica.
**Ejemplos:**
1. Enmascarar el tráfico saliente desde la LAN.
   ```bash
   iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE
   ```

#### 10. `-j` (jump to target)
**Estructura:**
```bash
iptables -A [CHAIN] [options] -j [target]
```
**Descripción:**
Especifica la acción que se tomará si un paquete coincide con la regla.
**Uso:**
Se utiliza para definir qué hacer con los paquetes que coinciden con las reglas.
**Ejemplos:**
1. Aceptar paquetes.
   ```bash
   iptables -A INPUT -s 192.168.106.200 -j ACCEPT
   ```
2. Rechazar paquetes.
   ```bash
   iptables -A INPUT -s 192.168.106.200 -j REJECT
   ```

#### 11. `-m state` y `--state`
**Estructura:**
```bash
iptables -A [CHAIN] -m state --state [state] -j [target]
```
**Descripción:**
Coincide con el estado de la conexión de los paquetes (NEW, ESTABLISHED, RELATED).
**Uso:**
Se utiliza para aplicar reglas basadas en el estado de la conexión.
**Ejemplos:**
1. Permitir el paso de paquetes cuya conexión se encuentra establecida.
   ```bash
   iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
   ```

### Ejemplo Completo de Reglas de `iptables`

A continuación, se presenta un conjunto completo de reglas `iptables` basado en los ejercicios abordados:

```bash
echo -n "Aplicando Reglas de Firewall..."

# FLUSH de reglas
iptables -F
iptables -X
iptables -Z
iptables -t nat -F

# Políticas por defecto
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

# Permitir tráfico loopback
iptables -A INPUT -i lo -j ACCEPT

# Permitir conexiones establecidas y relacionadas
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# Aceptar todos los paquetes que se originan desde 192.168.106.200
iptables -A INPUT -s 192.168.106.200 -j ACCEPT

# Rechazar todos los paquetes que se originan desde 192.168.106.200
iptables -A INPUT -s 192.168.106.200 -j REJECT

# Rechazar todos los paquetes que se originan desde 192.168.0.0
iptables -A INPUT -s 192.168.0.0/16 -j REJECT

# Permitir acceso al servidor web (puerto 80)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Permitir acceso al servidor FTP (puerto 20 y 21)
iptables -A INPUT -p tcp --dport 20 -j ACCEPT
iptables -A INPUT -p tcp --dport 21 -

j ACCEPT

# Permitir SSH desde 192.168.106.200
iptables -A INPUT -p tcp -s 192.168.106.200 --dport 22 -j ACCEPT

# Rechazar Telnet desde 192.168.106.200
iptables -A INPUT -p tcp -s 192.168.106.200 --dport 23 -j REJECT

# Rechazar todo el tráfico hacia 192.168.0.0/24 desde eth0
iptables -A INPUT -i eth0 -s 0.0.0.0/0 -d 192.168.0.0/24 -j REJECT

# Cerrar el rango de puertos bien conocidos
iptables -A INPUT -p tcp --dport 1:1023 -j REJECT
iptables -A INPUT -p udp --dport 1:1023 -j REJECT

# Aceptar tráfico HTTP desde la red 192.168.0.0/24
iptables -A FORWARD -s 192.168.0.0/24 -p tcp --dport 80 -j ACCEPT

# Aceptar tráfico HTTPS desde la red 192.168.0.0/24
iptables -A FORWARD -s 192.168.0.0/24 -p tcp --dport 443 -j ACCEPT

# Aceptar consultas DNS desde la red 192.168.0.0/24
iptables -A FORWARD -s 192.168.0.0/24 -p tcp --dport 53 -j ACCEPT
iptables -A FORWARD -s 192.168.0.0/24 -p udp --dport 53 -j ACCEPT

# Denegar todo el tráfico hacia la red 192.168.0.0/24
iptables -A FORWARD -d 192.168.0.0/24 -j REJECT

# Permitir tráfico SMTP, POP3 e IMAP para enviar y recibir emails
iptables -A INPUT -p tcp --dport 25 -j ACCEPT
iptables -A INPUT -p tcp --dport 587 -j ACCEPT
iptables -A INPUT -p tcp --dport 993 -j ACCEPT
iptables -A INPUT -p tcp --dport 995 -j ACCEPT

# Rechazar tráfico desde 192.168.3.0/24 a 192.168.2.0/24
iptables -A FORWARD -s 192.168.3.0/24 -d 192.168.2.0/24 -j REJECT

# Permitir tráfico TCP y UDP desde 192.168.3.5 a 192.168.0.5 (puerto 5432) y su respuesta
iptables -A FORWARD -p tcp -s 192.168.3.5 -d 192.168.0.5 --dport 5432 -j ACCEPT
iptables -A FORWARD -p udp -s 192.168.3.5 -d 192.168.0.5 --dport 5432 -j ACCEPT
iptables -A FORWARD -p tcp -s 192.168.0.5 -d 192.168.3.5 --sport 5432 -j ACCEPT
iptables -A FORWARD -p udp -s 192.168.0.5 -d 192.168.3.5 --sport 5432 -j ACCEPT

echo "Reglas de Firewall aplicadas."
```

Con esta síntesis, tienes una guía completa y rigurosa de los comandos más relevantes y repetidos en `iptables`, explicando su uso, estructura y ejemplos específicos basados en los ejercicios abordados. Esta información te preparará adecuadamente para un examen académico exigente sobre la configuración y gestión de firewalls con `iptables`.