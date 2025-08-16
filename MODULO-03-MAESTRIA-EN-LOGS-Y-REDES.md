# Módulo 3: Maestría en Logs y Redes

**Objetivo:** Los problemas más complejos rara vez se resuelven mirando solo los procesos locales. Se esconden en los logs y en el comportamiento de la red. Este módulo te da las herramientas para convertirte en un detective experto en estas dos áreas cruciales.

---

###  strumento `lnav` (The Log File Navigator)
Una verdadera joya. Es un visor de logs avanzado que va mucho más allá de `tail -f` y `less`. Analiza formatos de log automáticamente, fusiona archivos en una vista cronológica y te permite filtrar y analizar con una potencia increíble.

* **Por qué usarlo:** Transforma la tarea reactiva y tediosa de mirar logs en una experiencia de depuración interactiva y eficiente. Su capacidad para entender formatos y fusionar archivos de log te ahorra horas.
* **Reemplaza a:** `tail`, `less`, `grep`, `awk`, `sed` para el análisis de logs.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y lnav
    ```

#### Ejercicios Prácticos con `lnav`

* **Ejercicio 1: Investigación de un Error en Todo el Sistema**
    * **Escenario:** Un servicio ha fallado, pero no estás seguro de dónde se originó el problema. Quieres ver todos los logs del sistema (syslog, auth, kern, etc.) en una sola vista cronológica para entender el contexto completo.
    * **Comando:**
        ```bash
        sudo lnav /var/log
        ```
    * **Análisis del Resultado:** `lnav` escaneará todos los archivos en `/var/log`, detectará sus formatos y te presentará una vista unificada y coloreada, ordenada por fecha y hora. Errores en rojo, advertencias en amarillo. Ahora puedes ver si un error de kernel (`kern.log`) ocurrió justo antes de una falla de autenticación (`auth.log`), algo casi imposible de correlacionar con `tail`.

* **Ejercicio 2: Filtrado Interactivo en Tiempo Real**
    * **Escenario:** Estás sufriendo un ataque de fuerza bruta en tu servidor SSH. Quieres ver en tiempo real todas las conexiones fallidas.
    * **Comando:** (Dentro de la interfaz de `lnav`)
        1.  Inicia con `sudo lnav /var/log/auth.log`.
        2.  Presiona la tecla `:` para entrar al modo comando.
        3.  Escribe `filter-in failed password` y presiona `Enter`.
    * **Análisis del Resultado:** La vista se filtrará para mostrarte únicamente las líneas que contienen "failed password". Lo más impresionante es que este filtro se mantiene activo para los nuevos logs que llegan. Verás las nuevas líneas de ataque aparecer en tiempo real. Para desactivar, usa `:filter-out failed password`.

* **Ejercicio 3: Realizar Consultas SQL Directamente Sobre tus Logs**
    * **Escenario:** Quieres saber qué dirección IP es la que más intentos de login fallidos ha realizado.
    * **Comando:** (Dentro de la interfaz de `lnav` mientras ves `auth.log`)
        1.  Presiona la tecla `;` (punto y coma).
        2.  Escribe la siguiente consulta SQL y presiona `Enter`:
        ```sql
        SELECT c_ip, count(*) AS total FROM log_messages WHERE log_line LIKE '%Failed password%' GROUP BY c_ip ORDER BY total DESC
        ```
    * **Análisis del Resultado:** ¡Acabas de ejecutar una consulta SQL sobre un archivo de texto plano! `lnav` parsea los logs y te permite consultarlos como si fueran una base de datos. El resultado será una tabla con las IPs y el número de intentos fallidos, identificando al atacante principal. Esta es una capacidad que te diferencia del 99% de los administradores de sistemas.

---

###  strumento `bandwhich`
Una utilidad de TTY que muestra el uso de ancho de banda de red, pero lo hace de una forma muy visual y, lo más importante, agrupado por proceso y conexión.

* **Por qué usarlo:** A diferencia de `iftop` (que muestra por conexión) o `nethogs` (que muestra por proceso), `bandwhich` te da una vista que combina lo mejor de ambos, mostrando qué proceso está hablando con qué dirección IP.
* **Reemplaza a:** `nethogs`, `iftop`.
* **Instalación en Ubuntu 24.04 LTS:**
    * Se instala mejor a través de `cargo`.
    ```bash
    sudo apt install -y cargo
    cargo install bandwhich
    export PATH="$HOME/.cargo/bin:$PATH"
    ```

#### Ejercicios Prácticos con `bandwhich`

* **Ejercicio 1: Monitorización Básica del Tráfico Saliente**
    * **Escenario:** Quieres tener una idea rápida de qué procesos en tu servidor están consumiendo ancho de banda ahora mismo.
    * **Comando:**
        ```bash
        sudo bandwhich
        ```
    * **Análisis del Resultado:** `bandwhich` desplegará una interfaz mostrando los procesos que están usando la red, las direcciones IP a las que se conectan y el uso de subida/bajada. Es una forma excelente de "sentir" el pulso de la red de tu servidor.

* **Ejercicio 2: Identificar Conexiones Inesperadas**
    * **Escenario:** Sospechas que un proceso puede estar "llamando a casa" o conectándose a un destino no deseado.
    * **Comando:**
        ```bash
        sudo bandwhich
        ```
        *Observa la columna de `Connections`.*
    * **Análisis del Resultado:** Al dejarlo corriendo, puedes inspeccionar las direcciones IP remotas. Si ves un proceso conocido (ej. `node`) conectándose a una IP extraña que no reconoces, `bandwhich` te da el punto de partida perfecto para una investigación de seguridad.

* **Ejercicio 3: No Resolver Nombres de Host para Ver IPs Puras**
    * **Escenario:** La resolución de DNS (buscar el nombre asociado a una IP) puede añadir ruido o lentitud. Quieres ver solo las direcciones IP para un análisis más rápido.
    * **Comando:**
        ```bash
        sudo bandwhich --no-resolve
        ```
    * **Desglose del Comando:**
        * `--no-resolve` o `-n`: Desactiva la resolución de nombres de dominio.
    * **Análisis del Resultado:** La salida es ahora mucho más rápida y limpia, mostrando solo las direcciones IP numéricas. Esto es muy útil cuando estás en una red con DNS lento o cuando quieres copiar y pegar IPs rápidamente para investigarlas en otras herramientas.

---

###  strumento `dog`
Un cliente de DNS para la línea de comandos. Es una alternativa moderna, coloreada y mucho más amigable a `dig` y `nslookup`.

* **Por qué usarlo:** La salida de `dig` es completa pero muy verbosa y difícil de leer. `dog` presenta la misma información de una forma clara, concisa y coloreada. Además, soporta protocolos modernos como DNS-over-TLS (DoT) y DNS-over-HTTPS (DoH) de forma nativa.
* **Reemplaza a:** `dig`, `nslookup`, `host`.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y dog
    ```

#### Ejercicios Prácticos con `dog`

* **Ejercicio 1: Consulta DNS Simple y Legible**
    * **Escenario:** Simplemente quieres saber la dirección IP (registro A) de un dominio.
    * **Comando:**
        ```bash
        dog google.com
        ```
    * **Análisis del Resultado:** `dog` te devuelve una respuesta limpia y coloreada, mostrando solo la información esencial: el tipo de registro (A), el dominio y la dirección IP. Compara esto con la verbosidad de `dig google.com`.

* **Ejercicio 2: Consultar un Tipo de Registro Específico**
    * **Escenario:** Necesitas verificar los registros de intercambio de correo (MX) para un dominio.
    * **Comando:**
        ```bash
        dog google.com MX
        ```
    * **Desglose del Comando:**
        * `MX`: Especifica el tipo de registro a consultar. Puedes usar `A`, `AAAA`, `MX`, `TXT`, `CNAME`, etc.
    * **Análisis del Resultado:** La salida te muestra una lista clara de los servidores de correo para `google.com` con su prioridad. De nuevo, mucho más fácil de interpretar que la salida de `dig`.

* **Ejercicio 3: Usar un Servidor DNS Específico (y Seguro)**
    * **Escenario:** Sospechas que tu servidor DNS local te está dando respuestas incorrectas o cacheadas. Quieres consultar directamente a un servidor DNS público y seguro como el de Cloudflare (1.1.1.1) usando DNS-over-TLS.
    * **Comando:**
        ```bash
        dog google.com @tls://1.1.1.1
        ```
    * **Desglose del Comando:**
        * `@tls://1.1.1.1`: Le dice a `dog` que envíe la consulta a `1.1.1.1` usando el protocolo seguro DNS-over-TLS.
    * **Análisis del Resultado:** Has realizado una consulta DNS encriptada desde la línea de comandos con una sintaxis trivial. Hacer esto con `dig` es mucho más complejo. Esta es una característica clave para la administración de sistemas moderna y orientada a la seguridad.
