### **Bitácora de Configuración de HTTPS en Raspberry Pi**

**Fecha de inicio:** 15 de noviembre de 2024

---

#### **1. Configuración inicial del servidor web**

- Se configuró **Apache** como servidor web en la Raspberry Pi.
- Se estableció `/var/www/html` como el directorio raíz del servidor web.

---

#### **2. Habilitación de HTTPS en Apache**

- **Módulo SSL de Apache**: Verificamos que el módulo `ssl` estuviera habilitado en Apache usando:
  ```bash
  sudo a2enmod ssl
  ```
  Confirmamos que ya estaba habilitado.

- **Configuración de VirtualHost para HTTPS**:
  - Creamos o editamos el archivo de configuración `/etc/apache2/sites-available/default-ssl.conf`.
  - Configuramos el bloque `<VirtualHost *:443>` para servir contenido en HTTPS desde `/var/www/html` usando un certificado y clave SSL.

- **Certificado SSL autofirmado**:
  - Generamos un certificado SSL autofirmado como una solución temporal para habilitar HTTPS en el servidor local.
  - Comandos utilizados para generar el certificado:
    ```bash
    sudo mkdir -p /etc/ssl/localcerts
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/localcerts/selfsigned.key -out /etc/ssl/localcerts/selfsigned.crt
    ```
  - Configuramos Apache para usar este certificado en `/etc/apache2/sites-available/default-ssl.conf`.

- **Redirección de HTTP a HTTPS**:
  - Para asegurar que todo el tráfico HTTP se redirigiera automáticamente a HTTPS, editamos el archivo `/etc/apache2/sites-available/000-default.conf`, añadiendo las siguientes líneas para redirigir a HTTPS:
    ```plaintext
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
    ```

---

#### **3. Problema: Advertencia de seguridad en el navegador**

- Al acceder al sitio mediante HTTPS (`https://luthadel.local`), el navegador mostró una advertencia de seguridad debido al certificado autofirmado.
- La advertencia era esperada, ya que los certificados autofirmados no son confiables para los navegadores sin intervención manual.

**Soluciones exploradas:**
- **Añadir una excepción de seguridad en el navegador**: Se intentó la opción de añadir una excepción de seguridad temporal en el navegador.
- **Importación manual del certificado**: Se sugirió la opción de importar el certificado manualmente en el navegador o el sistema operativo para evitar advertencias recurrentes.

---

#### **4. Implementación de `mkcert` para certificados de desarrollo confiables**

- **Instalación de `mkcert` en la Raspberry Pi**:
  - Descargamos `mkcert` y lo instalamos en la Raspberry Pi para generar un certificado que los navegadores pudieran considerar de confianza en entornos locales.
  - Comandos utilizados:
    ```bash
    wget https://dl.filippo.io/mkcert/latest?for=linux/arm -O mkcert
    chmod +x mkcert
    sudo mv mkcert /usr/local/bin/
    mkcert -install
    ```

- **Generación del certificado con `mkcert`**:
  - Ejecutamos `mkcert` para crear un certificado para `luthadel.local` y `192.168.0.19`.
    ```bash
    mkcert luthadel.local 192.168.0.19
    ```
  - Archivos generados:
    - Certificado: `luthadel.local+1.pem`
    - Clave privada: `luthadel.local+1-key.pem`

- **Configuración de Apache para usar el certificado `mkcert`**:
  - Movimos los archivos generados a `/etc/ssl/localcerts/` y configuramos Apache para usarlos.
    ```bash
    sudo mv luthadel.local+1.pem /etc/ssl/localcerts/luthadel.crt
    sudo mv luthadel.local+1-key.pem /etc/ssl/localcerts/luthadel.key
    ```

  - Actualizamos `/etc/apache2/sites-available/default-ssl.conf` para apuntar al nuevo certificado:
    ```plaintext
    SSLCertificateFile /etc/ssl/localcerts/luthadel.crt
    SSLCertificateKeyFile /etc/ssl/localcerts/luthadel.key
    ```

---

#### **5. Resolución de advertencia de certificado en el navegador**

- Aunque `mkcert` generó un certificado confiable, algunos navegadores aún mostraban advertencias.
- **Solución final**: Importamos manualmente el certificado raíz de `mkcert` en el navegador para eliminar las advertencias de seguridad de forma definitiva. La ubicación del certificado raíz fue:
  ```plaintext
  ~/.local/share/mkcert/rootCA.pem
  ```
- Procedimiento:
  - Importamos `rootCA.pem` en la configuración de certificados del navegador en la pestaña **Autoridades** y seleccionamos la opción para confiar en el certificado.

---

### **6. Verificación Final**

- Realizamos la prueba final accediendo a `https://luthadel.local` y `https://192.168.0.19`.
- Confirmamos que HTTPS estaba activo y que el navegador aceptaba el certificado sin mostrar advertencias de seguridad, indicando que el proceso de configuración de HTTPS fue exitoso.

---

### **Estado Final**

- **HTTPS**: Activo y confiable en el entorno local.
- **Certificado**: Emitido por `mkcert` y reconocido por el navegador tras importar el certificado raíz.
- **Redirección**: HTTP redirige automáticamente a HTTPS.

**Notas adicionales**:
- La importación manual del certificado raíz solo es necesaria en entornos locales. En entornos de producción, se recomienda obtener un certificado SSL de una CA oficial como Let’s Encrypt.
