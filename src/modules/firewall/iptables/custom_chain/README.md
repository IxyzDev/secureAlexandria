Una cadena personalizada en `iptables` es una cadena (chain) que ha sido creada por el usuario para organizar y gestionar reglas de firewall de manera más modular y específica. A diferencia de las cadenas predeterminadas (INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING) que vienen integradas en `iptables`, las cadenas personalizadas son definidas por el administrador del sistema para propósitos específicos.

### ¿Por qué utilizar cadenas personalizadas?

1. **Organización:** Permiten una mejor organización y gestión de reglas complejas, dividiéndolas en partes lógicas.
2. **Reusabilidad:** Las reglas comunes a múltiples contextos pueden ser colocadas en una cadena personalizada y referenciadas desde múltiples lugares.
3. **Mantenimiento:** Facilitan el mantenimiento de reglas al permitir cambios en un solo lugar que afectan múltiples cadenas.

### Cómo crear y utilizar cadenas personalizadas

#### Crear una cadena personalizada

Para crear una cadena personalizada, se utiliza el comando `-N` seguido del nombre de la nueva cadena.

**Ejemplo:**
```bash
iptables -N CUSTOM_CHAIN
```
**Descripción:** Crea una nueva cadena llamada `CUSTOM_CHAIN`.

#### Añadir reglas a una cadena personalizada

Una vez creada la cadena, se pueden añadir reglas a ella de la misma manera que se añaden a las cadenas predeterminadas.

**Ejemplo:**
```bash
iptables -A CUSTOM_CHAIN -s 192.168.0.0/24 -j ACCEPT
iptables -A CUSTOM_CHAIN -s 10.0.0.0/8 -j DROP
```
**Descripción:** 
- La primera regla acepta todos los paquetes provenientes de la subred `192.168.0.0/24`.
- La segunda regla descarta todos los paquetes provenientes de la red `10.0.0.0/8`.

#### Llamar a una cadena personalizada desde una cadena predeterminada

Para utilizar la cadena personalizada, se puede "saltar" a ella desde una de las cadenas predeterminadas usando el objetivo `-j` seguido del nombre de la cadena personalizada.

**Ejemplo:**
```bash
iptables -A INPUT -p tcp --dport 80 -j CUSTOM_CHAIN
```
**Descripción:** Esta regla en la cadena `INPUT` dirige todo el tráfico TCP destinado al puerto 80 a la cadena `CUSTOM_CHAIN`.

### Ejemplo Completo

A continuación se muestra un ejemplo completo de cómo crear y utilizar una cadena personalizada:

```bash
# Crear una cadena personalizada llamada CUSTOM_CHAIN
iptables -N CUSTOM_CHAIN

# Añadir reglas a la cadena personalizada
iptables -A CUSTOM_CHAIN -s 192.168.0.0/24 -j ACCEPT
iptables -A CUSTOM_CHAIN -s 10.0.0.0/8 -j DROP

# Llamar a la cadena personalizada desde la cadena INPUT
iptables -A INPUT -p tcp --dport 80 -j CUSTOM_CHAIN
```

### Limpieza de cadenas personalizadas

Cuando ya no se necesita una cadena personalizada, se debe eliminar utilizando el comando `-X` después de haber eliminado todas las referencias a ella.

**Ejemplo:**
```bash
# Limpiar las reglas en la cadena personalizada
iptables -F CUSTOM_CHAIN

# Eliminar la cadena personalizada
iptables -X CUSTOM_CHAIN
```

**Descripción:**
- `iptables -F CUSTOM_CHAIN` limpia todas las reglas de `CUSTOM_CHAIN`.
- `iptables -X CUSTOM_CHAIN` elimina la cadena `CUSTOM_CHAIN`.

### Resumen

**Estructura:** 
```bash
iptables -N [CHAIN_NAME]  # Crear
iptables -A [CHAIN_NAME] [rules]  # Añadir reglas
iptables -A [CHAIN] -j [CHAIN_NAME]  # Referenciar
iptables -F [CHAIN_NAME]  # Limpiar
iptables -X [CHAIN_NAME]  # Eliminar
```

**Uso:** 
Se utilizan para organizar mejor las reglas de firewall, aumentar la modularidad, facilitar el mantenimiento y la reutilización de reglas en `iptables`.

**Ejemplos Prácticos:**
- Creación de una cadena personalizada para reglas comunes de acceso web.
- Reutilización de una cadena personalizada en múltiples contextos para mantener la consistencia.