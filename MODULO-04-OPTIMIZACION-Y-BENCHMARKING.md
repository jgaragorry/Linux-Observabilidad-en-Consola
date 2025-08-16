# Módulo 4: Optimización y Benchmarking

**Objetivo:** Un administrador de sistemas senior no solo arregla problemas, sino que los previene y optimiza los flujos de trabajo. Este módulo te enseña a medir el rendimiento de forma científica, a automatizar búsquedas y a mejorar tu interacción diaria con la consola.

---

###  strumento `hyperfine`
Una herramienta de benchmarking para la línea de comandos. Te permite comparar la velocidad de ejecución de dos o más comandos de forma rigurosa y estadística.

* **Por qué usarlo:** El comando `time` solo ejecuta algo una vez. `hyperfine` lo ejecuta múltiples veces, calienta la caché, detecta anomalías estadísticas y te da un veredicto claro sobre qué comando es más rápido y por cuánto.
* **Reemplaza a:** `time` y a las suposiciones.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y hyperfine
    ```

#### Ejercicios Prácticos con `hyperfine`

* **Ejercicio 1: Demostrar que `fd` es más rápido que `find`**
    * **Escenario:** Quieres probarle a tu equipo que vale la pena instalar `fd` porque es más rápido que el `find` tradicional para buscar archivos.
    * **Comando:**
        ```bash
        hyperfine 'find . -name "*.log"' 'fd ".log$"'
        ```
    * **Desglose del Comando:**
        * `'comando1' 'comando2'`: `hyperfine` toma los comandos a comparar como argumentos separados. Las comillas simples son importantes para que los caracteres especiales no sean interpretados por el shell.
    * **Análisis del Resultado:** `hyperfine` ejecutará ambos comandos varias veces y te presentará una tabla comparativa. Concluirá con una frase como: `Summary: 'fd ".log$"' ran 15.32 ± 2.41 times faster than 'find . -name "*.log"'`. Ahora tienes datos, no opiniones.

* **Ejercicio 2: Calentamiento de Caché para Benchmarks Justos**
    * **Escenario:** Estás comparando dos comandos que leen mucho del disco (como `grep` vs `ripgrep`). La primera ejecución siempre es más lenta por la caché del disco. Quieres eliminar ese factor.
    * **Comando:**
        ```bash
        hyperfine --warmup 3 'grep -r "error" /var/log' 'rg "error" /var/log'
        ```
    * **Desglose del Comando:**
        * `--warmup 3`: Ejecuta cada comando 3 veces *antes* de empezar a medir. Esto asegura que la caché del sistema de archivos esté "caliente" y las mediciones sean sobre el rendimiento del programa, no del disco.
    * **Análisis del Resultado:** Los resultados serán más consistentes y representativos del rendimiento real de los programas en condiciones normales de operación, donde los archivos accedidos frecuentemente ya están en caché.

* **Ejercicio 3: Exportar Resultados para Documentación**
    * **Escenario:** Has realizado un benchmark importante para decidir qué herramienta usar en un script de producción. Quieres guardar los resultados en un formato estándar.
    * **Comando:**
        ```bash
        hyperfine --export-markdown benchmark.md 'comando1' 'comando2'
        ```
    * **Desglose del Comando:**
        * `--export-markdown benchmark.md`: Exporta los resultados a un archivo Markdown llamado `benchmark.md`. También soporta `--export-csv` y `--export-json`.
    * **Análisis del Resultado:** Se creará un archivo `benchmark.md` con una tabla bien formateada de los resultados, lista para ser pegada en la documentación de tu proyecto, en un pull request o en un informe.

---

###  strumento `fd` (fdfind)
Una alternativa simple, rápida e intuitiva a `find`. Está escrito en Rust y optimizado para la velocidad.

* **Por qué usarlo:** La sintaxis de `find` es notoriamente compleja y fácil de olvidar. `fd` tiene una sintaxis mucho más lógica, es más rápido por defecto (ignora archivos ocultos y `.gitignore`) y tiene una salida coloreada.
* **Reemplaza a:** `find`.
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y fd-find
    # Ubuntu lo instala como 'fdfind'. Creamos un enlace simbólico para usar 'fd'.
    sudo ln -s $(which fdfind) /usr/local/bin/fd
    ```

#### Ejercicios Prácticos con `fd`

* **Ejercicio 1: Búsqueda Simple por Nombre de Archivo**
    * **Escenario:** Quieres encontrar todos los archivos de configuración de Nginx en el sistema.
    * **Comando:**
        ```bash
        fd nginx.conf /etc
        ```
    * **Desglose del Comando:**
        * `nginx.conf`: El patrón a buscar (acepta expresiones regulares básicas por defecto).
        * `/etc`: El directorio donde buscar.
    * **Análisis del Resultado:** `fd` encontrará rápidamente el archivo. La sintaxis es `fd <patrón> [directorio]`, mucho más simple que `find /etc -name nginx.conf`.

* **Ejercicio 2: Encontrar Archivos por Extensión y Ejecutar un Comando**
    * **Escenario:** Quieres encontrar todos los archivos `.tmp` en tu directorio home y eliminarlos.
    * **Comando:**
        ```bash
        fd -e tmp --exec rm
        ```
    * **Desglose del Comando:**
        * `-e tmp`: Busca archivos por extensión (`-e`xtension).
        * `--exec rm`: Por cada resultado encontrado, ejecuta el comando `rm`. `fd` pasará el nombre del archivo como argumento.
    * **Análisis del Resultado:** Esta es la forma moderna y segura de `find . -name "*.tmp" -delete`. La sintaxis es más legible y menos propensa a errores.

* **Ejercicio 3: Búsqueda Interactiva de Archivos**
    * **Escenario:** Sabes que quieres editar un archivo, pero no recuerdas su nombre o ubicación exacta. Quieres buscarlo y abrirlo en tu editor inmediatamente.
    * **Comando:**
        * *Necesitas `fzf` instalado: `sudo apt install fzf`*
        ```bash
        fd . | fzf | xargs nano
        ```
    * **Análisis del Resultado:** Este es un patrón extremadamente poderoso. `fd .` lista todos los archivos rápidamente. `| fzf` envía esa lista a `fzf`, un buscador difuso interactivo que te permite escribir partes del nombre y ver los resultados filtrados al instante. Al presionar `Enter` en `fzf`, `xargs nano` toma el archivo seleccionado y lo abre en `nano`. Es la forma más rápida de encontrar y abrir un archivo en toda la TTY.

---

###  strumento `bat` (batcat)
Un `cat` con superpoderes. Muestra el contenido de archivos con resaltado de sintaxis, numeración de líneas, integración con `git` y paginación automática.

* **Por qué usarlo:** Mejora drásticamente la legibilidad de scripts, archivos de configuración y código fuente en la terminal. Una vez que lo usas, `cat` te parecerá aburrido y primitivo.
* **Reemplaza a:** `cat`, `less` (para visualización).
* **Instalación en Ubuntu 24.04 LTS:**
    ```bash
    sudo apt install -y bat
    # Ubuntu lo instala como 'batcat'. Creamos un enlace simbólico para usar 'bat'.
    sudo ln -s $(which batcat) /usr/local/bin/bat
    ```

#### Ejercicios Prácticos con `bat`

* **Ejercicio 1: Visualización Mejorada de un Archivo de Configuración**
    * **Escenario:** Vas a revisar tu archivo `~/.bashrc`.
    * **Comando:**
        ```bash
        bat ~/.bashrc
        ```
    * **Análisis del Resultado:** En lugar de un texto plano y monótono, verás tu `.bashrc` con los comentarios en un color, las variables en otro, los comandos en otro, etc. Además, si el archivo es largo, `bat` lo paginará automáticamente a través de `less`, por lo que obtienes lo mejor de ambos mundos.

* **Ejercicio 2: Mostrar Cambios de Git en un Archivo**
    * **Escenario:** Estás dentro de un repositorio de Git y has modificado un archivo. Quieres ver el contenido del archivo y resaltar visualmente las líneas que has añadido o modificado.
    * **Comando:**
        ```bash
        bat mi_script.py
        ```
    * **Análisis del Resultado:** `bat` detecta que estás en un repositorio Git y mostrará en la barra lateral izquierda los símbolos `+` para las líneas añadidas y `~` para las modificadas, dándote un "diff" visual directamente en el visor de archivos.

* **Ejercicio 3: Usar `bat` como Paginar de `man`**
    * **Escenario:** Las páginas del manual de Linux son útiles pero visualmente aburridas. Quieres que se vean tan bien como los archivos que ves con `bat`.
    * **Comando:**
        * Añade esto a tu `~/.bashrc` o `~/.zshrc`:
        ```bash
        export MANPAGER="sh -c 'col -bx | bat -l man -p'"
        ```
        * Luego, abre una nueva terminal o ejecuta `source ~/.bashrc` y prueba:
        ```bash
        man ls
        ```
    * **Análisis del Resultado:** ¡La página del manual de `ls` ahora está coloreada y formateada por `bat`! Los títulos, las flags y las secciones están resaltados, haciendo la lectura mucho más agradable y eficiente. Este es un ejemplo perfecto de cómo las herramientas modernas pueden componerse para mejorar todo tu entorno de trabajo.
