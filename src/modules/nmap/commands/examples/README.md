2`nmap` es una herramienta extremadamente versátil y poderosa utilizada para el descubrimiento de redes y la auditoría de seguridad. Ofrece una amplia gama de tipos de escaneo, cada uno con propósitos y técnicas específicos. Aquí te proporciono una lista de los tipos de escaneo más comunes que puedes realizar con `nmap`:

1. **Escaneo de Ping (Ping Scan)**:
   - Comprueba si los hosts están activos sin escanear puertos.
   - Comando: `nmap -sn [target]`

2. **Escaneo de Puertos TCP SYN (SYN Scan)**:
   - Escaneo rápido y el más popular, también conocido como "half-open scanning".
   - Comando: `nmap -sS [target]`

3. **Escaneo de Puertos TCP Connect (Connect Scan)**:
   - Utiliza llamadas al sistema operativo para abrir una conexión completa, similar a cómo lo haría un usuario.
   - Comando: `nmap -sT [target]`

4. **Escaneo de Puertos UDP (UDP Scan)**:
   - Escanea los puertos UDP, que son menos comunes pero importantes para servicios como DNS y SNMP.
   - Comando: `nmap -sU [target]`

5. **Escaneo FIN (FIN Scan)**:
   - Envía paquetes FIN para tratar de evadir firewalls y detectar puertos abiertos en sistemas que no están configurados para manejar paquetes FIN de manera estándar.
   - Comando: `nmap -sF [target]`

6. **Escaneo Xmas (Xmas Scan)**:
   - Envía paquetes con las banderas FIN, URG y PUSH activadas, esperando respuestas inusuales para identificar sistemas operativos y configuraciones.
   - Comando: `nmap -sX [target]`

7. **Escaneo Null (Null Scan)**:
   - Envía un paquete sin banderas TCP establecidas; los sistemas Unix responden de una manera que permite identificar puertos abiertos.
   - Comando: `nmap -sN [target]`

8. **Escaneo de Detección de Versión (Version Detection)**:
   - Intenta determinar qué servicios están corriendo en los puertos abiertos y qué versión tienen.
   - Comando: `nmap -sV [target]`

9. **Escaneo de Scripts (Script Scan)**:
   - Utiliza scripts de NSE (Nmap Scripting Engine) para detectar vulnerabilidades, configuraciones erróneas y otras características de seguridad.
   - Comando: `nmap --script=[category] [target]`

10. **Escaneo de IP Protocol Scan**:
    - Explora para determinar qué protocolos IP (TCP, UDP, ICMP, etc.) son soportados por los hosts.
    - Comando: `nmap -sO [target]`

11. **Escaneo ACK (ACK Scan)**:
    - Envía paquetes ACK para determinar si un puerto está filtrado o no, útil para mapear reglas de firewall.
    - Comando: `nmap -sA [target]`

12. **Escaneo SCTP INIT (SCTP INIT Scan)**:
    - Escanea usando paquetes SCTP para identificar si los puertos están abiertos o cerrados.
    - Comando: `nmap -sY [target]`

13. **Escaneo Maimon (Maimon Scan)**:
    - Envía paquetes TCP con las banderas FIN/ACK, basado en una observación del comportamiento peculiar de ciertos tipos de hosts.
    - Comando: `nmap -sM [target]`

Cada tipo de escaneo es adecuado para diferentes situaciones y objetivos de seguridad. Seleccionar el tipo correcto depende de lo que necesites descubrir o probar en tu red. Además, siempre es crucial utilizar `nmap` de manera ética y legal, preferiblemente en redes que tienes permiso para analizar.