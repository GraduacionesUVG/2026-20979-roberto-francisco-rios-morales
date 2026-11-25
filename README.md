# Desarrollo de un algoritmo genético simple para la búsqueda de configuraciones eficientes de semáforos con simulaciones en zona 1 de Ciudad de Guatemala

Trabajo de graduación de Ingeniería en Ciencias de la Computación y Tecnologías de la Información,
Universidad del Valle de Guatemala, 2026.

## Descripción

El tránsito vehicular en la zona 1 de Ciudad de Guatemala se congestiona con facilidad, y los tiempos
de los semáforos influyen directamente en cuánto tarda un vehículo en cruzar la red. Este proyecto
desarrolla un **algoritmo genético simple (AGS)** que busca una configuración eficiente de los semáforos
de esa zona: la duración del ciclo de cada intersección, el reparto del verde entre sus fases y el
desfase entre intersecciones.

Cómo funciona:

1. La red vial de la zona 1 se importó de OpenStreetMap al simulador de tránsito **Eclipse SUMO**.
2. La demanda de prueba es un conjunto fijo de 2,002 vehículos con rutas predefinidas. Ese nivel lleva
   la red a su capacidad y se determinó con el diagrama fundamental macroscópico de la red
   (`saturacion.py`).
3. Cada individuo del algoritmo es un cromosoma de 440 genes que codifica el plan semafórico completo.
   El controlador (`controller.py`) lo decodifica en una red de SUMO y la simula sin teletransportes.
4. La aptitud se calcula con los tiempos de viaje y de espera promedio que reporta SUMO, mediante
   J = viaje + 1.1 · espera. El algoritmo busca minimizar J.
5. Selección por torneo, cruce uniforme, mutación aleatoria y elitismo, con PyGAD. Las simulaciones de
   cada generación corren en paralelo.

Resultados principales, contra la configuración que netconvert asigna al importar la red (ciclo de 90 s,
sin desfases):

- El tiempo de viaje promedio bajó **42.53 %** y el tiempo de espera promedio bajó **62.34 %**.
- La función objetivo bajó **51.2 %** (J de 878.1 s a 428.8 s, en la generación 126 de 150).
- Simulada de nuevo con 35 combinaciones de demanda y semilla, la solución superó al punto de partida en
  todas, con una mediana de reducción de 40.3 %, y no trabó la red en ninguna.

## Repositorio de desarrollo

La investigación y el desarrollo del proyecto se hicieron en el repositorio
[robertriosm/upmega](https://github.com/robertriosm/upmega). Ahí está el historial completo del trabajo
(49 commits): prototipos, pruebas, experimentos con el algoritmo, análisis de resultados y la generación
del video explicativo.

La carpeta [`src/`](src/) de este repositorio es una copia del código de `upmega` en su último commit
(`ff9a8e8`, del 24 de noviembre de 2026), con un solo cambio posterior: `controller.py` y
`video/grabar_sumo.py` ubican SUMO mediante la variable de entorno `SUMO_HOME` en lugar de una ruta fija
(ver [Notas de portabilidad](#notas-de-portabilidad)).

## Tecnologías Utilizadas

- **Python 3.13**: lenguaje de todo el proyecto.
- **Eclipse SUMO 1.23.1**: simulación microscópica de tránsito (`sumo`, `sumo-gui`, `netconvert`,
  `polyconvert`, `duarouter`).
- **TraCI y sumolib**: control de SUMO desde Python y lectura de redes.
- **PyGAD 3.5.0**: motor del algoritmo genético.
- **NumPy, pandas y Matplotlib**: análisis de resultados y gráficas.
- **cloudpickle**: serialización de las corridas del algoritmo.
- **OpenStreetMap**: fuente del mapa de la zona 1 (`map.osm`).
- **Jupyter Notebook**: cuadernos de análisis (`analisis.ipynb`, `cifras_complementarias.ipynb`).
- **ManimGL y FFmpeg**: video explicativo del algoritmo (carpeta `src/video/`, opcional).

## Requisitos Previos

- **Windows 10 u 11**: es el sistema en el que se desarrolló y probó el proyecto.
- **Python 3.13** (la versión con la que se hicieron las corridas) y `pip`.
- **Eclipse SUMO 1.23.1**, en cualquier carpeta de instalación. Descarga:
  <https://sumo.dlr.de/docs/Downloads.php>
- La variable de entorno **`SUMO_HOME`**, que apunta a la instalación de SUMO. El código la usa para
  encontrar los ejecutables de SUMO y el instalador de Windows la define automáticamente; ver
  [`src/.env.example`](src/.env.example).
- **Git**.
- Recomendado: un procesador de 6 núcleos físicos o más. La evaluación en paralelo deja de acelerar a
  partir de 6 procesos.

## Instalación

1. Clonar el repositorio y entrar al código fuente:

   ```bash
   git clone https://github.com/GraduacionesUVG/2026-20979-roberto-francisco-rios-morales.git
   cd 2026-20979-roberto-francisco-rios-morales/src
   ```

2. Crear un entorno virtual e instalar las dependencias:

   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   pip install -r requirements.txt
   ```

   > **Sobre `package.json`:** el proyecto está escrito en Python, no en Node.js, así que no lleva
   > `package.json`. Sus dependencias, con versiones fijas, están en
   > [`src/requirements.txt`](src/requirements.txt), que es el archivo equivalente en Python. Así lo
   > indica la [FAQ de la guía de entrega](https://csuvg.github.io/Guia-entrega-trabajo-graduacion/docs/faq.html):
   > un proyecto que no es de Node.js no necesita `package.json`, pero debe incluir el archivo de
   > dependencias de su tecnología (`requirements.txt` en el caso de Python).

3. Verificar la variable de entorno `SUMO_HOME`. El controlador la usa para ubicar `sumo` y, si no está
   definida o no apunta a una instalación válida, se detiene con un mensaje que lo explica. El proyecto no
   usa credenciales ni lee un archivo `.env`; [`src/.env.example`](src/.env.example) documenta la única
   variable necesaria.

   ```powershell
   echo $env:SUMO_HOME
   # Si sale vacío, definirla con la carpeta donde se instaló SUMO y abrir una terminal nueva:
   setx SUMO_HOME "C:\Program Files (x86)\Eclipse\Sumo"
   ```

4. Comprobar la instalación:

   ```bash
   python main.py --help
   ```

## Ejecución

Todos los comandos se ejecutan desde `src/` con el entorno virtual activado.

### 1. Ver la red en SUMO

```bash
sumo-gui -c map.sumo.cfg
```

Abre la red de la zona 1 con los edificios del mapa ([captura 01](capturas/captura-01.png)).

### 2. Correr el algoritmo genético

```bash
python main.py --num-of-cars 2600 --workers 6
```

- `--num-of-cars 2600` genera la demanda de la tesis: 2,600 vehículos solicitados a `randomTrips`, de
  los que quedan 2,002 con ruta válida. La primera vez se crea `demand/cars2600_s42.rou.xml`; la semilla
  42 hace que la demanda sea reproducible.
- `--workers` define cuántas simulaciones corren en paralelo.
- `--saturation K` detiene la corrida si el mejor fitness no mejora en K generaciones seguidas.
- La configuración de la tesis está en `main.py`: 150 generaciones, población de 30, 12 padres, torneo de
  5, cruce uniforme, mutación de 2 % y semilla 42. Son 4,380 simulaciones y la corrida tarda varias horas.
- En cada generación se imprime el mejor fitness ([captura 05](capturas/captura-05.png)). Al terminar se
  abre la gráfica de evolución de PyGAD, y la corrida termina cuando se cierra esa ventana.

La corrida deja, con el nombre `g150p30m12touuniP2c2600s42`:

| Archivo | Contenido |
|---|---|
| `{corrida}.meta.json` | Parámetros, versiones, métricas del punto de partida y de la mejor solución |
| `{corrida}.evaluations.csv` | Todas las evaluaciones: id, generación, fitness, tiempos medios de viaje y de espera, tiempo perdido, vehículos sin llegar y si la red se trabó |
| `{corrida}.pkl` | Instancia de PyGAD con la población final |
| `logics/{id}.net.xml` | Red de SUMO de cada solución evaluada (la mejor se indica en `meta.json`) |
| `detail/` | Salidas detalladas de SUMO del punto de partida y de la mejor solución |

Para ver la mejor solución en SUMO-GUI:

```bash
sumo-gui -n logics/<id-de-la-mejor>.net.xml -r demand/cars2600_s42.rou.xml
```

### 3. Análisis de resultados

`saturacion.py` corre sus propias simulaciones. Los demás scripts leen los archivos de una corrida
completa, así que se ejecutan después del paso 2. Los cuadernos `.ipynb` requieren instalar Jupyter aparte
(`pip install notebook`).

| Comando | Qué hace |
|---|---|
| `python saturacion.py` | Diagrama fundamental de la red para elegir la demanda ([captura 04](capturas/captura-04.png)) |
| `python analisis_corrida.py g150p30m12touuniP2c2600s42` | Cifras del capítulo de resultados (`{corrida}.analisis.json`) |
| `python graficas.py g150p30m12touuniP2c2600s42` | Comparación de demoras netconvert contra AGS ([captura 10](capturas/captura-10.png)) |
| `python mapa_aristas.py g150p30m12touuniP2c2600s42` | Mapa del cambio de tiempo perdido por calle ([captura 11](capturas/captura-11.png)) |
| `python robustez.py g150p30m12touuniP2c2600s42` | Robustez de la solución ante otras semillas de demanda y de SUMO |
| `python cromosoma.py g150p30m12touuniP2c2600s42` | Estructura y decodificación del cromosoma (`{corrida}.cromosoma.txt`) |
| `python aceleracion.py g150p30m12touuniP2c2600s42 --lote 6 --workers 1 6 3 1` | Aceleración de la evaluación en paralelo |

El repositorio ya incluye los resultados ligeros de la corrida de la tesis: los `.json`, `.csv` y `.png`
con el prefijo `g150p30m12touuniP2c2600s42`, y las carpetas `robustez/` y `saturacion/`. Los archivos
pesados (`.pkl`, `logics/`, `detail/`, `outputs/` y `history/`, unos 16 GB) no se versionan y se regeneran
con el paso 2.

### Notas de portabilidad

El algoritmo y los scripts de análisis encuentran SUMO con la variable `SUMO_HOME`
(`controller.py`, método `get_sumo_binary`, y `video/grabar_sumo.py`), así que no dependen de la carpeta
de instalación. Las únicas rutas absolutas que quedan en el código no intervienen en la ejecución del
algoritmo:

- `video/custom_config.yml` (`ffmpeg_bin`): FFmpeg para renderizar el video explicativo. La forma de
  actualizarla está en [`src/video/README.md`](src/video/README.md).
- `diagramas.py` y `mapa_aristas.py` (`DESTINO`): carpeta donde se guardaban las figuras para el
  documento LaTeX de la tesis.

## Capturas

Las 11 capturas del sistema en funcionamiento están en [/capturas](capturas/), en el orden de uso:

| # | Captura | Descripción |
|---|---|---|
| 01 | [captura-01.png](capturas/captura-01.png) | Red completa de la zona 1 importada de OpenStreetMap y cargada en SUMO-GUI (`map.sumo.cfg`). |
| 02 | [captura-02.png](capturas/captura-02.png) | Detalle de una intersección semaforizada en SUMO-GUI, con carriles y estado de las señales. |
| 03 | [captura-03.png](capturas/captura-03.png) | Opciones de línea de comandos del programa principal (`python main.py --help`). |
| 04 | [captura-04.png](capturas/captura-04.png) | Diagrama fundamental de la red generado por `saturacion.py`: densidad crítica y demanda a la que la red se traba. |
| 05 | [captura-05.png](capturas/captura-05.png) | Ejecución corta del algoritmo genético en consola (2 generaciones, población 6) con el mejor fitness por generación. |
| 06 | [captura-06.png](capturas/captura-06.png) | Evolución del mejor individuo en la corrida de la tesis: J baja de 878.1 s a 428.8 s. |
| 07 | [captura-07.png](capturas/captura-07.png) | SUMO-GUI simulando la red con el plan de netconvert. En rojo, los vehículos detenidos (velocidad ≤ 0.1 m/s). |
| 08 | [captura-08.png](capturas/captura-08.png) | SUMO-GUI en el mismo instante y con la misma demanda, con la mejor solución del algoritmo: menos colas. |
| 09 | [captura-09.png](capturas/captura-09.png) | Comparación lado a lado de ambos planes, con vehículos detenidos y espera acumulada. |
| 10 | [captura-10.png](capturas/captura-10.png) | Demora por vehículo (media, mediana, p90, p99 y máximo) de netconvert contra la mejor solución, generada por `graficas.py`. |
| 11 | [captura-11.png](capturas/captura-11.png) | Mapa de la red con el cambio del tiempo perdido en cada calle, generado por `mapa_aristas.py`. |

## Demo

Video demostrativo: [/demo/demo.mp4](demo/demo.mp4)

## Documentación

- Informe final: [/docs/informe_final.pdf](docs/informe_final.pdf)
- Consentimiento de uso de foto y video firmado:
  [/docs/consentimiento_firmado.pdf](docs/consentimiento_firmado.pdf)

## Estructura del repositorio

```
2026-20979-roberto-francisco-rios-morales/
├── capturas/          Capturas del sistema (captura-01.png … captura-11.png)
├── demo/              Video demostrativo (demo.mp4)
├── docs/              Informe final y consentimiento firmado
├── src/               Código fuente (copia del repositorio de desarrollo upmega)
│   ├── main.py        Punto de entrada: corre el algoritmo genético
│   ├── sga.py         Algoritmo genético simple (PyGAD): cromosoma, operadores, registro
│   ├── controller.py  Interacción con SUMO: decodificación del cromosoma y evaluación
│   ├── demand.py      Generación de la demanda de vehículos
│   ├── *.py           Scripts de análisis (ver tabla de Ejecución)
│   ├── *.ipynb        Cuadernos de análisis
│   ├── map.*, *.net.xml, typemap.xml, osmNetconvert.typ.xml   Red y mapa de la zona 1
│   ├── robustez/, saturacion/   Resultados ligeros de la tesis
│   ├── video/         Generación del video explicativo con ManimGL
│   ├── requirements.txt   Dependencias de Python (equivale a package.json, que no aplica)
│   └── .env.example
├── LICENSE
└── README.md
```

## Autores

- Roberto Francisco Rios Morales - 20979

Asesor: PhD. Gabriel Antonio Barrientos Rodríguez

## Licencia

Este proyecto se distribuye bajo la licencia [MIT](LICENSE).

Excepciones:

- `src/randomTrips.py` es una herramienta de Eclipse SUMO y conserva su licencia original
  (EPL-2.0 o GPL-2.0-or-later), indicada en la cabecera del archivo.
- Los datos del mapa (`src/map.osm` y las redes derivadas) provienen de © colaboradores de
  OpenStreetMap, bajo la licencia [ODbL](https://opendatacommons.org/licenses/odbl/).
