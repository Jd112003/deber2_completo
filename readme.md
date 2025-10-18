# Deber 2 SO

# Juan Diego Chicaiza, Mauricio Mantilla y James Soto

## Estructura del Repositorio

El proyecto está organizado en las siguientes carpetas principales:

- `deber2_SO/`: Implementaciones del clásico problema de la cena de los filósofos.
- `Simulador-Scheduling/`: Un simulador de algoritmos de planificación de CPU.
- `sistemas-operativos-ejer03/`: Implementaciones del problema productor-consumidor.

A continuación se detalla el contenido de cada sección.

## Enlaces a READMEs internos

Para más detalles específicos de cada subproyecto, consulta los READMEs internos disponibles:

- [Simulador-Scheduling/experiments_report.md](Simulador-Scheduling/experiments_report.md)
- [deber2_SO/README.md](deber2_SO/README.md)
- [sistemas-operativos-ejer03/README.md](sistemas-operativos-ejer03/README.md)

### `deber2_SO` - La Cena de los Filósofos

Esta sección aborda el problema de la cena de los filósofos utilizando diferentes lenguajes de programación y modelos de concurrencia.

- **`filosofos_c/`**: Solución en C, separada en:
  - `hilos`: Implementación utilizando hilos (pthreads).
  - `procesos`: Implementación utilizando procesos y comunicación entre procesos (IPC).
- **`filosofos_java/`**: Solución en Java, con tres enfoques:
  - `hilos`: Implementación con hilos de Java.
  - `procesos`: Simulación de procesos.
  - `procesosreales`: Una implementación cliente-servidor que implementa procesos independientes.
- **`filosofos_python/`**: Solución en Python, dividida en:
  - `solucion_hilos`: Implementación con el módulo `threading`.
  - `solucion_procesos`: Implementación con el módulo `multiprocessing`.

### `Simulador-Scheduling`

Contiene un programa en C (`simulacion.c`) que simula varios algoritmos de planificación de CPU (ej. FCFS, SJF, Round Robin).

# Simulador de Algoritmos de Scheduling (C)

Este proyecto implementa un simulador discreto de planificación de CPU en C. Permite seleccionar entre tres políticas y comparar su desempeño bajo diferentes cargas de trabajo.

Archivo principal: [Simulador-Scheduling/simulacion.c](Simulador-Scheduling/simulacion.c)

## Requerimientos del trabajo

- a) Definir requerimientos para tres algoritmos:
  - FCFS (First-Come, First-Served)
  - Round Robin (RR) con quantum configurable
  - SJF (Shortest Job First) no expropiativo (usa cola de prioridad)
- b) Implementar el simulador en C.
- c) Permitir seleccionar la política y mostrar resultados.
- Comparar los algoritmos bajo tres escenarios:
  - 50% IO-bound / 50% CPU-bound
  - 90% IO-bound / 10% CPU-bound
  - 10% IO-bound / 90% CPU-bound
- Métricas: throughput, turnaround time y average response time.

## Algoritmos implementados

- FCFS: cola FIFO sin expropiación (simulado como RR con quantum muy grande).
- Round Robin: expropiativo por quantum configurable.
- SJF: selecciona el proceso con menor próximo CPU burst usando cola de prioridad (no expropiativo).

Selección en tiempo de ejecución vía [`main`](Simulador-Scheduling/simulacion.c).

## Modelo de simulación

- Proceso con ráfagas alternadas CPU/I-O:
  - Tipos: [`BurstType`](Simulador-Scheduling/simulacion.c) = {CPU_BURST, IO_BURST}
  - Proceso: [`Process`](Simulador-Scheduling/simulacion.c) con `arrival_time`, lista de [`Burst`](Simulador-Scheduling/simulacion.c), `state` = [`ProcessState`](Simulador-Scheduling/simulacion.c), y tiempos `start_time`/`finish_time`.
- Núcleo del simulador: [`Simulator`](Simulador-Scheduling/simulacion.c) con reloj discreto, colas READY/WAITING y política [`SchedulerPolicy`](Simulador-Scheduling/simulacion.c).
- Estructuras de colas:
  - FIFO: [`enqueue`](Simulador-Scheduling/simulacion.c), [`dequeue`](Simulador-Scheduling/simulacion.c)
  - Prioridad para SJF: [`priority_enqueue`](Simulador-Scheduling/simulacion.c), [`priority_dequeue`](Simulador-Scheduling/simulacion.c)

Ciclo por tick en [`run_simulation`](Simulador-Scheduling/simulacion.c):
1) Llegadas: [`move_arrivals`](Simulador-Scheduling/simulacion.c)
2) Dispositivos: [`update_waiting`](Simulador-Scheduling/simulacion.c)
3) CPU (y expropiación RR): [`update_running`](Simulador-Scheduling/simulacion.c)
4) Despacho si CPU libre: [`dispatch_if_idle`](Simulador-Scheduling/simulacion.c)

## Generación de workloads

Se generan 100 procesos con llegadas agrupadas (10 procesos por unidad de tiempo). Cada proceso tiene 10 bursts, con sesgo IO/CPU según el escenario:

- Generación: [`create_workload`](Simulador-Scheduling/simulacion.c) y [`generate_bursts`](Simulador-Scheduling/simulacion.c)
  - IO-bound: ~75% I/O, I/O largos (4–8), CPU cortos (1–3)
  - CPU-bound: ~75% CPU, CPU largos (6–12), I/O cortos (1–3)

Escenarios seleccionables en tiempo de ejecución:
- 90% IO-bound / 10% CPU-bound
- 50% IO-bound / 50% CPU-bound
- 10% IO-bound / 90% CPU-bound

## Métricas calculadas

En [`print_statistics`](Simulador-Scheduling/simulacion.c) se reporta por proceso y promedios:

- Turnaround por proceso: $T_i = finish_i - arrival_i$
- Response por proceso: $R_i = start_i - arrival_i$
- Promedios:
  $$
  \overline{T} = \frac{1}{n}\sum_i T_i
  \quad
  \overline{R} = \frac{1}{n}\sum_i R_i
  $$
- Throughput total:
  $$
  throughput = \frac{N_{completados}}{T_{sim}}
  $$

Nota: `total_wait_time` existe en [`Process`](Simulador-Scheduling/simulacion.c) pero no se usa en el cálculo impreso.

## Cómo ejecutar

Compilar y correr en la carpeta del proyecto:

```bash
gcc -O2 simulacion.c -o simulador
./simulador
```

El programa solicitará:
- Política (1=FCFS, 2=RR con quantum, 3=SJF)
- Workload (1=90/10 IO/CPU, 2=50/50, 3=10/90)

## Observaciones de diseño

- FCFS: sin expropiación (RR con quantum grande).
- RR: expropiación al agotar quantum; reencola en READY.
- SJF: selección por “próxima ráfaga de CPU” usando cola prioritaria; no expropiativo.
- Llegadas agrupadas por reloj para generar contención realista.
- Reloj discreto avanza 1 unidad por iteración.

Funciones clave:
- Entrada/salida principal: [`main`](Simulador-Scheduling/simulacion.c)
- Simulación: [`run_simulation`](Simulador-Scheduling/simulacion.c)
- Estadísticas: [`print_statistics`](Simulador-Scheduling/simulacion.c)
- Flujo de eventos: [`move_arrivals`](Simulador-Scheduling/simulacion.c), [`update_waiting`](Simulador-Scheduling/simulacion.c), [`update_running`](Simulador-Scheduling/simulacion.c), [`dispatch_if_idle`](Simulador-Scheduling/simulacion.c)

## Hipótesis preliminares (antes de ejecutar)

- 90% IO-bound: SJF debería minimizar $\overline{T}$ y $\overline{R}$; throughput alto por ráfagas CPU cortas.
- 50/50: SJF suele ganar en $\overline{T}$; RR mejora $\overline{R}$ para tareas largas frente a FCFS.
- 90% CPU-bound: RR mejora respuesta percibida; FCFS puede sufrir efecto “convoy”; SJF no expropiativo puede retrasar tareas largas.

Se recomienda ejecutar los tres escenarios con las tres políticas y comparar las métricas impresas por [`print_statistics`](Simulador-Scheduling/simulacion.c).

### `sistemas-operativos-ejer03` - Productor-Consumidor

Esta carpeta contiene soluciones al problema del productor-consumidor.

- **`C/`**: Implementaciones en C usando hilos y procesos.
- **`Java/`**: Implementación en Java.
- **`Python/`**: Implementaciones en Python para hilos y procesos.

## Cómo Compilar y Ejecutar

Cada sub-proyecto tiene su propia estructura. Para instrucciones detalladas, por favor revise los archivos `README.md` o `Makefile` dentro de cada carpeta específica.

- **Proyectos en C**: Generalmente se compilan con `make` y se ejecutan desde la terminal.
- **Proyectos en Java**: Se compilan con `javac` y se ejecutan con `java`.
- **Proyectos en Python**: Se ejecutan directamente con el intérprete de Python (ej. `python3 script.py`).

## Pregunta 4 — Cambios al Kernel (Resumen)

- Propuesta: “Sistema de logs mejorado” en Linux que permite filtrar por nivel (INFO/WARN/…) y por PID, y expone estadísticas por proceso vía sysfs/proc.
- Entorno de pruebas: QEMU ARM64 para iteración rápida y Raspberry Pi para validación en hardware real.
- Entregables: objetivos, diseño mínimo viable, fases de implementación y plan de pruebas.

Documento completo: [ejer04propuesta.md](ejer04propuesta.md)
