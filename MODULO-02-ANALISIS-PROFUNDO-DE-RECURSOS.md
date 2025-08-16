# Módulo 2: Análisis Profundo de Recursos

**Objetivo:** Cuando un dashboard del Módulo 1 te indica que "algo" está mal (CPU alta, disco lleno), necesitas herramientas especializadas para hacer un zoom y entender el "porqué". Este módulo te da las lupas y microscopios para la investigación forense de recursos.

---

###  strumento `gdu` (Go Disk Usage)
Un analizador de uso de disco increíblemente rápido. Escanea directorios y te presenta una interfaz interactiva para navegar y liberar espacio.

* **Por qué usarlo:** Es órdenes de magnitud más rápido que `du -sh *` o `ncdu` gracias a su procesamiento en paralelo. Su interfaz interactiva te permite explorar el árbol de directorios y eliminar archivos/carpetas directamente.
* **Reemplaza a:** `du`, `ncdu`.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y gdu
    ```

#### Ejercicios Prácticos con `gdu`

* **Ejercicio 1: Encontrar los "Cerdos" del Espacio en Disco**
    * **Escenario:** Recibes una alerta de "disco casi lleno" en `/var`. Necesitas encontrar rápidamente qué subdirectorio está ocupando más espacio.
    * **Comando:**
        ```bash
        sudo gdu /var
        ```
    * **Desglose del Comando:**
        * `sudo`: Necesario para escanear directorios del sistema sin errores de permisos.
        * `gdu /var`: Inicia el escaneo en el directorio `/var`.
    * **Análisis del Resultado:** En segundos, `gdu` te presentará una lista de los directorios dentro de `/var` ordenados por tamaño. Probablemente `/var/log` o `/var/lib/docker` estarán en la cima. Puedes usar las flechas y `Enter` para navegar dentro de ellos y seguir investigando.

* **Ejercicio 2: Liberar Espacio de Forma Segura e Interactiva**
    * **Escenario:** Has identificado en `/var/log` unos archivos de log rotados (`.gz`, `.1`) que son antiguos y ocupan gigabytes. Quieres borrarlos.
    * **Comando:** (Dentro de la interfaz de `gdu`)
        1.  Navega hasta el directorio que contiene los archivos a eliminar.
        2.  Usa las flechas para seleccionar un archivo o directorio.
        3.  Presiona la tecla `d`.
        4.  `gdu` te pedirá confirmación. Presiona `y`.
    * **Análisis del Resultado:** El archivo o directorio se elimina y el espacio se recalcula al instante. Este flujo de trabajo (analizar -> navegar -> eliminar) dentro de una sola herramienta es mucho más seguro y eficiente que tener `du` en una terminal y `rm` en otra.

* **Ejercicio 3: Exportar un Análisis para Reportarlo**
    * **Escenario:** No tienes permisos para borrar archivos, pero necesitas enviarle un reporte a tu superior sobre el estado del disco.
    * **Comando:**
        ```bash
        sudo gdu -o /tmp/analisis_disco.json /var
        ```
    * **Desglose del Comando:**
        * `-o /tmp/analisis_disco.json`: Le dice a `gdu` que no inicie la interfaz interactiva, sino que exporte los resultados del escaneo a un archivo JSON.
    * **Análisis del Resultado:** Se creará el archivo `/tmp/analisis_disco.json`. Este archivo puede ser enviado, archivado o procesado por otros scripts. Es una forma excelente de realizar análisis de disco programados (vía `cron`) y guardar los resultados para un análisis de tendencias a lo largo del tiempo.

---

###  strumento `zenith`
Otro monitor de sistema, pero con un superpoder: gráficos históricos. Mientras que `btm` te muestra el "ahora", `zenith` te muestra los últimos 60 segundos, lo cual es crucial para entender patrones y picos de uso.

* **Por qué usarlo:** Su capacidad para mostrar gráficos de uso de CPU, red y disco a lo largo del tiempo en la TTY es única. Es la herramienta perfecta para diagnosticar problemas intermitentes.
* **Reemplaza a:** `btm` o `htop` cuando necesitas contexto histórico.
* **Instalación en Ubuntu 24.04 LTS:**
    * Zenith se instala mejor a través de `cargo`, el gestor de paquetes de Rust.
    ```bash
    sudo apt install -y cargo
    cargo install zenith
    # Añade el directorio de cargo a tu PATH si no está ya
    export PATH="$HOME/.cargo/bin:$PATH"
    ```

#### Ejercicios Prácticos con `zenith`

* **Ejercicio 1: Visualizar el Impacto de un Proceso en el Tiempo**
    * **Escenario:** Vas a compilar un programa o a ejecutar un backup y quieres ver el impacto que tiene en la CPU y el disco a lo largo de su ejecución.
    * **Comando:**
        ```bash
        zenith
        ```
    * **Análisis del Resultado:** Mientras tu otro proceso se ejecuta, observa los gráficos de CPU y Disco en `zenith`. Verás cómo las líneas suben y se mantienen, dándote una idea clara del perfil de recursos del proceso. Si ves picos y valles, indica un comportamiento de I/O intensivo; si la CPU se mantiene alta, es un proceso de cómputo.

* **Ejercicio 2: Identificar Picos de Tráfico de Red**
    * **Escenario:** Los usuarios se quejan de lentitud en la red "a veces". Sospechas que hay picos de tráfico.
    * **Comando:** (Dentro de la interfaz de `zenith`)
        1.  Usa la tecla `Tab` para moverte entre los paneles.
        2.  Enfócate en el panel de Red (Network).
    * **Análisis del Resultado:** El gráfico te mostrará el uso de la red en los últimos 60 segundos. Deja `zenith` corriendo y espera a que ocurra el problema. Un pico repentino en el gráfico de Recepción (Rx) o Transmisión (Tx) confirmará tu sospecha visualmente, permitiéndote correlacionar el pico con la hora del reporte.

* **Ejercicio 3: Profundizar en el Uso de la GPU**
    * **Escenario:** Estás en un servidor con una GPU NVIDIA y ejecutas tareas de IA/ML. Necesitas monitorizar el uso de la GPU.
    * **Comando:**
        ```bash
        zenith
        ```
    * **Análisis del Resultado:** Si `zenith` detecta una GPU compatible (NVIDIA con drivers instalados), mostrará automáticamente un panel de GPU con su uso, uso de memoria y temperatura. Esto es extremadamente útil, ya que tradicionalmente requiere el uso de `nvidia-smi`, y `zenith` lo integra en un dashboard general.

---

###  strumento `iotop`
Es el `top` para las operaciones de I/O (Entrada/Salida) del disco. Te dice exactamente qué proceso está leyendo o escribiendo en el disco y a qué velocidad.

* **Por qué usarlo:** Cuando la carga del sistema (`load average`) es alta pero el uso de CPU es bajo, el culpable casi siempre es el disco. `iotop` es la única herramienta que te dice quién es el culpable.
* **Reemplaza a:** Adivinar qué proceso está ralentizando el disco.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y iotop
    ```

#### Ejercicios Prácticos con `iotop`

* **Ejercicio 1: Encontrar al Proceso que "Secuestra" el Disco**
    * **Escenario:** El LED del disco duro no para de parpadear y todas las aplicaciones responden lento.
    * **Comando:**
        ```bash
        sudo iotop
        ```
    * **Análisis del Resultado:** `iotop` te mostrará una lista de procesos ordenados por su uso de I/O. El proceso en la cima, con valores altos en las columnas `DISK READ` o `DISK WRITE`, es tu culpable. Probablemente sea un proceso de base de datos, un backup, o el propio `journald` escribiendo logs.

* **Ejercicio 2: Mostrar Solo los Procesos que Están Usando el Disco**
    * **Escenario:** La salida de `iotop` es muy ruidosa, con muchos procesos que no hacen nada. Quieres una vista más limpia.
    * **Comando:**
        ```bash
        sudo iotop -o
        ```
    * **Desglose del Comando:**
        * `-o` o `--only`: Muestra únicamente los procesos o hilos que están realizando operaciones de I/O en este momento.
    * **Análisis del Resultado:** La vista es ahora mucho más limpia. Solo aparecerán procesos cuando realmente lean o escriban en el disco, haciendo mucho más fácil identificar la actividad relevante.

* **Ejercicio 3: Ver el Uso Acumulado de I/O**
    * **Escenario:** Quieres saber qué proceso ha escrito más datos en el disco desde que iniciaste el monitoreo, no solo el uso instantáneo.
    * **Comando:**
        ```bash
        sudo iotop -a
        ```
    * **Desglose del Comando:**
        * `-a` o `--accumulated`: Muestra el I/O acumulado en lugar de la velocidad.
    * **Análisis del Resultado:** Las columnas ahora muestran la cantidad total de datos leídos/escritos por cada proceso desde que `iotop` se inició. Esto es perfecto para identificar "fugas" de datos o procesos que, aunque no son intensivos en un momento dado, a lo largo del tiempo escriben una cantidad enorme de información al disco.
