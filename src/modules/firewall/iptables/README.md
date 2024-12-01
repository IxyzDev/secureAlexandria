### README: Configuración de Reglas de Firewall con `iptables`

Este documento tiene como objetivo proporcionar una comprensión clara de las reglas de firewall utilizando `iptables`, cómo implementarlas, y ofrecer ejemplos prácticos para futuras configuraciones. 

### Índice
- [README: Configuración de Reglas de Firewall con `iptables`](#readme-configuración-de-reglas-de-firewall-con-iptables)
- [Índice](#índice)
- [Introducción a `iptables`](#introducción-a-iptables)
- [Estructura de las Reglas de `iptables`](#estructura-de-las-reglas-de-iptables)
- [Reglas Prácticas de Ejemplo](#reglas-prácticas-de-ejemplo)
- [Configuraciones Avanzadas](#configuraciones-avanzadas)
- [Conclusión](#conclusión)

### Introducción a `iptables`

`iptables` es una herramienta para administrar las reglas de filtrado de paquetes en sistemas basados en Linux. Funciona con diferentes tablas y cadenas para controlar el tráfico de red.

**Tablas principales:**
- `filter`: Para el filtrado estándar de paquetes.
- `nat`: Para la traducción de direcciones de red.
- `mangle`: Para la manipulación especial de paquetes.
- `raw`: Para controlar la conexión de seguimiento (stateful inspection).

### Estructura de las Reglas de `iptables`

Cada regla de `iptables` tiene una estructura que incluye la tabla, la cadena, las condiciones y la acción a tomar.

**Estructura básica:**
```bash
iptables -t <tabla> -A <cadena> [opciones] -j <objetivo>
```

**Explicación de los parámetros:**
- `-t <tabla>`: Tabla a utilizar (`filter`, `nat`, `mangle`, `raw`).
- `-A <cadena>`: Cadena a la que se añade la regla (`INPUT`, `OUTPUT`, `FORWARD`, etc.).
- `[opciones]`: Filtros adicionales, como protocolos, direcciones IP, puertos, etc.
- `-j <objetivo>`: Acción a tomar (`ACCEPT`, `DROP`, `DNAT`, etc.).

### Reglas Prácticas de Ejemplo

1. **DNAT (Destination NAT) para Redireccionar Tráfico:**
   - Redireccionar el tráfico de la IP pública a servidores internos.
```bash
# Redirige el tráfico SSH (puerto 22) a un servidor interno específico
iptables -t nat -A PREROUTING -d 200.14.67.201 -p tcp --dport 22 -j DNAT --to-destination 192.168.22.1:22

# Redirige el tráfico SMTP (puerto 25) a otro servidor interno
iptables -t nat -A PREROUTING -d 200.14.67.201 -p tcp --dport 25 -j DNAT --to-destination 192.168.22.2:25

# Redirige el tráfico HTTP (puerto 80) a un servidor web interno
iptables -t nat -A PREROUTING -d 200.14.67.201 -p tcp --dport 80 -j DNAT --to-destination 192.168.22.3:80
```

2. **Filtrar Direcciones IP Específicas:**
   - Bloquea tráfico desde ciertas subredes.
```bash
# Bloquea tráfico de la subred 192.168.0.0/24 en la interfaz eth1
iptables -A INPUT -i eth1 -s 192.168.0.0/24 -j DROP

# Bloquea tráfico de la subred 10.0.0.0/8 en la interfaz eth1
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP

# Bloquea tráfico de la subred 172.168.0.0/16 en la interfaz eth1
iptables -A INPUT -i eth1 -s 172.168.0.0/16 -j DROP
```

3. **Permitir Tráfico Solo desde una Subred Específica:**
   - Permite tráfico desde una subred concreta y bloquea otros.
```bash
# Permite tráfico desde 192.168.1.0/24 a través de la interfaz eth0
iptables -A INPUT -i eth0 -s 192.168.1.0/24 -j ACCEPT

# Bloquea todo el tráfico que no sea desde 192.168.1.0/24
iptables -A INPUT -i eth0 -s 0.0.0.0/0 -j DROP
```

4. **Reglas para Cadenas Específicas:**
   - Añade reglas personalizadas a cadenas específicas.
```bash
# Permitir todo el tráfico en la cadena OUTPUT
iptables -A OUTPUT -j ACCEPT

# Permitir tráfico de HTTP/HTTPS desde la red interna
iptables -A FORWARD -p tcp --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp --dport 443 -j ACCEPT
```

### Configuraciones Avanzadas

1. **Masquerading (SNAT):**
   - Traduce la dirección IP de origen para permitir la navegación en internet desde una red privada.
```bash
# Habilita masquerading para la red privada (Ejemplo con eth0 como interfaz de salida)
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j MASQUERADE

# Otro ejemplo usando la interfaz ppp0
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o ppp0 -j MASQUERADE
```

2. **Rate Limiting:**
   - Limita el número de conexiones permitidas por segundo para evitar ataques de denegación de servicio.
```bash
# Permite solo 5 conexiones SSH por segundo
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP

# Limita solicitudes ICMP (ping) a 1 por segundo
iptables -A INPUT -p icmp -m limit --limit 1/second -j ACCEPT
iptables -A INPUT -p icmp -j DROP
```

3. **Logging (Registro):**
   - Registra eventos para ayudar en el análisis de seguridad.
```bash
# Registra todos los intentos fallidos de conexión SSH
iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH-ATTEMPT: "

# Registra todos los intentos de conexión HTTP
iptables -A INPUT -p tcp --dport 80 -j LOG --log-prefix "HTTP-ATTEMPT: "
```

4. **Port Knocking:**
   - Un método para abrir puertos cerrados utilizando una secuencia de paquetes.
```bash
# Cerramos inicialmente el puerto 22
iptables -A INPUT -p tcp --dport 22 -j DROP

# Crea la secuencia de knocking
iptables -N KNOCK1
iptables -N KNOCK2
iptables -N KNOCK3
iptables -A INPUT -p tcp --dport 1234 -m recent --name KNOCK1 --set -j DROP
iptables -A INPUT -p tcp --dport 2345 -m recent --name KNOCK1 --rcheck --seconds 10 -j KNOCK2
iptables -A KNOCK2 -p tcp --dport 2345 -m recent --name KNOCK2 --set -j DROP
iptables -A INPUT -p tcp --dport 3456 -m recent --name KNOCK2 --rcheck --seconds 10 -j KNOCK3
iptables -A KNOCK3 -p tcp --dport 3456 -m recent --name KNOCK3 --set -j DROP
iptables -A INPUT -p tcp --dport 22 -m recent --name KNOCK3 --rcheck --seconds 10 -j ACCEPT
```

5. **SNAT (Source NAT):**
   - Cambia la dirección IP de origen de los paquetes que salen de una red.
```bash
# Cambia la IP de origen de la red 192.168.1.0/24 a una IP pública específica
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 203.0.113.1
```

6. **Forwarding entre Interfaces:**
   - Permite el reenvío de tráfico entre diferentes interfaces.
```bash
# Permite el reenvío desde eth1 a eth0
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

# Permite el reenvío desde eth0 a eth1
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```

### Conclusión

`iptables` proporciona una manera poderosa de controlar el tráfico de red. Ya sea que estés redirigiendo tráfico, bloqueando IPs específicas o creando un firewall avanzado, estas reglas básicas te proporcionan un punto de partida sólido.

**Consejos Adicionales:**
- Guarda tus reglas para evitar perderlas tras un reinicio:
  ```bash
 

 iptables-save > /etc/iptables/rules.v4
  ip6tables-save > /etc/iptables/rules.v6
  ```
  
- Restaura las reglas guardadas:
  ```bash
  iptables-restore < /etc/iptables/rules.v4
  ip6tables-restore < /etc/iptables/rules.v6
  ```

