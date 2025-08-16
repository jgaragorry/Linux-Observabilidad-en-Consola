# Módulo 1: Observabilidad Instantánea

**Objetivo:** Obtener una visión completa y en tiempo real del estado de salud de tu sistema y contenedores. Estas herramientas reemplazan la necesidad de ejecutar `top`, `ps aux`, `df`, `free` y `docker stats` en múltiples ventanas, dándote un centro de mando unificado.

---

###  strumento `btm` (Bottom)
Es un monitor de recursos gráfico, rápido e increíblemente interactivo. Es lo que `top` y `htop` desearían ser.

* **Por qué usarlo:** Proporciona una densidad de información superior a `htop` en una sola pantalla (CPU, Mem, Disco, Red, Procesos) y su interactividad (filtrado, ordenado, árbol de procesos) es mucho más fluida.
* **Reemplaza a:** `top`, `htop`.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt update && sudo apt install -y bottom
    ```

#### Ejercicios Prácticos con `btm`

* **Ejercicio 1: Diagnóstico Rápido de un "Servidor Lento"**
    * **Escenario:** Los usuarios reportan lentitud. Necesitas una vista de 360 grados del sistema en menos de 5 segundos.
    * **Comando:**
        ```bash
        btm
        ```
    * **Desglose del Comando:**
        * `btm`: Ejecuta la herramienta con su vista por defecto.
    * **Análisis del Resultado:** Observa inmediatamente los widgets de CPU, Memoria, Red y Procesos. ¿Hay algún núcleo de CPU al 100%? ¿La memoria SWAP está siendo utilizada intensivamente? ¿Hay un pico inusual de tráfico de red? `btm` te da la respuesta a estas tres preguntas en una sola vista, algo imposible con `top`.

* **Ejercicio 2: Encontrar y Eliminar un Proceso Zombi o Colgado**
    * **Escenario:** Sospechas que una aplicación ha dejado un proceso hijo colgado que consume recursos. Necesitas encontrarlo por su relación padre-hijo.
    * **Comando:** (Dentro de la interfaz de `btm`)
        1.  Presiona `t` para activar la vista de árbol de procesos.
        2.  Usa las flechas para navegar hasta el proceso sospechoso.
        3.  Presiona `k` para enviar la señal `SIGTERM` y eliminarlo. Si no funciona, presiona `k` de nuevo, y cuando te pida la señal, escribe `9` y `Enter` para enviar `SIGKILL`.
    * **Análisis del Resultado:** La vista de árbol (`t`) es una de las características más potentes. Te permite entender la jerarquía de procesos sin necesidad de ejecutar `pstree`. Puedes ver qué script o servicio ha lanzado un proceso problemático, facilitando un diagnóstico mucho más profundo.

* **Ejercicio 3: Aislar la Actividad de un Usuario o Aplicación Específica**
    * **Escenario:** Quieres monitorizar exclusivamente los procesos que pertenecen al usuario `www-data` porque sospechas de un problema en el servidor web.
    * **Comando:** (Dentro de la interfaz de `btm`)
        1.  Presiona `/` para abrir la barra de búsqueda/filtro.
        2.  Escribe `www-data`.
        3.  Presiona `Enter`.
    * **Análisis del Resultado:** La lista de procesos se filtra instantáneamente para mostrar solo aquellos que coinciden. Ahora puedes ver el consumo de CPU y memoria exclusivamente de los procesos de tu servidor web en tiempo real. Esto es inmensamente más rápido y claro que hacer `ps aux | grep www-data` repetidamente. Para limpiar el filtro, presiona `/` de nuevo y borra el texto.

---

###  strumento `procs`
Un reemplazo moderno para `ps` con una salida coloreada, legible y con información adicional por defecto (como puertos TCP/UDP y lectura/escritura de disco).

* **Por qué usarlo:** La sintaxis de búsqueda es más intuitiva que `ps` y `grep`. Proporciona más contexto (contenedores, puertos) sin necesidad de usar otros comandos como `lsof` o `netstat`.
* **Reemplaza a:** `ps`.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y procs
    ```

#### Ejercicios Prácticos con `procs`

* **Ejercicio 1: Búsqueda Lógica de Procesos**
    * **Escenario:** Necesitas encontrar todos los procesos que pertenezcan al usuario `root` **Y** que contengan `systemd` en su nombre.
    * **Comando:**
        ```bash
        procs --and root systemd
        ```
    * **Desglose del Comando:**
        * `--and`: Actúa como un operador lógico `Y`. Todos los términos siguientes deben coincidir.
        * `root`, `systemd`: Los términos de búsqueda.
    * **Análisis del Resultado:** `procs` te devuelve una lista limpia solo con los procesos que cumplen ambas condiciones. Hacer esto con `ps` implicaría algo como `ps aux | grep root | grep systemd`, que es menos eficiente y propenso a errores. `procs` también soporta `--or`.

* **Ejercicio 2: Ver Qué Procesos Están Escuchando en Puertos de Red**
    * **Escenario:** Quieres ver rápidamente qué aplicaciones tienen puertos abiertos en el servidor, sin usar `netstat` o `ss`.
    * **Comando:**
        ```bash
        procs --tcp --udp
        ```
    * **Desglose del Comando:**
        * `--tcp`: Añade una columna que muestra los puertos TCP en los que el proceso está escuchando.
        * `--udp`: Añade una columna para los puertos UDP.
    * **Análisis del Resultado:** Obtienes una lista de todos los procesos y, junto a los relevantes (como `sshd`, `nginx`, etc.), verás inmediatamente el puerto que tienen asignado (ej: `*:22`). Es una fusión de `ps` y `netstat -tulnp` en un solo comando legible.

* **Ejercicio 3: Inspeccionar Procesos Dentro de un Contenedor Docker**
    * **Escenario:** Estás corriendo contenedores y quieres ver los procesos del host, pero también saber a qué contenedor pertenece cada uno.
    * **Comando:**
        ```bash
        procs --docker
        ```
    * **Desglose del Comando:**
        * `--docker`: Agrega una columna "Docker" a la salida.
    * **Análisis del Resultado:** Para cualquier proceso que se esté ejecutando dentro de un contenedor, la nueva columna te mostrará el nombre del contenedor. Esto es oro puro para la observabilidad en entornos de microservicios, ya que te da contexto inmediato sin tener que hacer `docker ps` y luego `docker top <container_id>`.

---

###  strumento `ctop`
Un `top` para el mundo de los contenedores. Proporciona una vista en tiempo real de las métricas de múltiples contenedores (Docker, Podman, etc.).

* **Por qué usarlo:** Ofrece un dashboard centralizado para todos tus contenedores, mostrando CPU, memoria, red y disco en tiempo real. Es mucho más práctico que ejecutar `docker stats` para cada contenedor individualmente.
* **Reemplaza a:** `docker stats`.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo snap install ctop
    ```

#### Ejercicios Prácticos con `ctop`

* **Ejercicio 1: Vista de Pájaro del Consumo de Recursos de Contenedores**
    * **Escenario:** Tienes múltiples contenedores corriendo y necesitas saber cuál de ellos está consumiendo más CPU o memoria en este preciso momento.
    * **Comando:**
        ```bash
        sudo ctop
        ```
    * **Análisis del Resultado:** `ctop` presenta una lista de todos tus contenedores activos con su estado y métricas clave. Puedes ordenar por cualquier columna (presiona `s` para cambiar el criterio de ordenamiento) para traer al frente inmediatamente al contenedor más "ruidoso".

* **Ejercicio 2: Inspeccionar los Logs de un Contenedor en Tiempo Real**
    * **Escenario:** Un contenedor parece estar fallando. Quieres ver sus logs sin tener que salir de la interfaz de monitoreo.
    * **Comando:** (Dentro de la interfaz de `ctop`)
        1.  Usa las flechas para seleccionar el contenedor problemático.
        2.  Presiona `l` (de logs).
    * **Análisis del Resultado:** La pantalla cambiará para mostrar un `tail -f` de los logs del contenedor seleccionado. Esto agiliza enormemente el ciclo de "observar un problema -> investigar la causa", ya que no necesitas abrir otra terminal y escribir `docker logs -f <container_name>`. Para volver, presiona `Esc`.

* **Ejercicio 3: Gestionar el Ciclo de Vida de los Contenedores**
    * **Escenario:** Has identificado un contenedor que necesita ser reiniciado. Quieres hacerlo directamente desde la herramienta de monitoreo.
    * **Comando:** (Dentro de la interfaz de `ctop`)
        1.  Selecciona el contenedor que deseas gestionar.
        2.  Presiona `a` (attach) para entrar al shell del contenedor, o...
        3.  Presiona `s` para detenerlo (Stop).
        4.  Presiona `r` para reiniciarlo (Restart).
    * **Análisis del Resultado:** `ctop` no es solo una herramienta de visualización, es una herramienta de gestión. La capacidad de detener, reiniciar o incluso entrar a un contenedor desde la misma interfaz donde detectaste el problema es un ahorro de tiempo y cambio de contexto masivo.
