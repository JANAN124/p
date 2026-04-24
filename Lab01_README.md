<div align="center">
<picture>
    <source srcset="https://imgur.com/5bYAzsb.png" media="(prefers-color-scheme: dark)">
    <source srcset="https://imgur.com/Os03JoE.png" media="(prefers-color-scheme: light)">
    <img src="https://imgur.com/Os03JoE.png" alt="Escudo UNAL" width="300px">
</picture>

<h3>Curso de Robótica 2026-I</h3>
<h1>Laboratorio No. 1</h1>
<h2>Robótica Industrial: Trayectorias, entradas y salidas digitales</h2>
<h3>ABB IRB 140 · RobotStudio</h3>

<h4>Profesores: Pedro Fabián Cárdenas Herrera · Manuel Felipe Carranza Montenegro</h4>
<h4>Estudiantes: Janan Libardo Carreño Riaño · Cristian Stiven Hoyos Peralta · Jose Andres Zapata Piñeros</h4>

<p>
  <img alt="ABB IRB140" src="https://img.shields.io/badge/Robot-ABB%20IRB%20140-CC0000?logoColor=white">
  <img alt="RobotStudio" src="https://img.shields.io/badge/Software-RobotStudio-41C9D0?logoColor=white">
  <img alt="RAPID" src="https://img.shields.io/badge/Lenguaje-RAPID-FF6600?logoColor=white">
  <img alt="Estado" src="https://img.shields.io/badge/Estado-Completo-brightgreen">
</p>
</div>

---

<div align="center">
  <img src="imagenes/Robot_ABB_Portada.png" alt="Robot ABB IRB 140" width="450px">
  <br><em>Celda de manufactura y robot manipulador ABB IRB 140</em>
</div>

---

## Tabla de contenido

1. [Introducción y descripción de la solución](#1-introducción-y-descripción-de-la-solución)
2. [Diagrama de flujo de acciones del robot](#2-diagrama-de-flujo-de-acciones-del-robot)
3. [Plano de planta](#3-plano-de-planta)
4. [Descripción de las funciones utilizadas](#4-descripción-de-las-funciones-utilizadas)
5. [Arquitectura y Modelado de la Herramienta](#5-arquitectura-y-modelado-de-la-herramienta)
6. [Lógica y Programación en RAPID](#6-lógica-y-programación-en-rapid)
7. [Evidencias Multimedia](#7-evidencias-multimedia)

---

## 1. Introducción y descripción de la solución

### 1.1 Planteamiento del problema

Este laboratorio aborda la automatización de un proceso inspirado en la industria de alimentos, enfocado específicamente en la decoración de una torta virtual. El objetivo principal es sustituir el trabajo manual por un sistema programado que garantice mayor precisión, control y repetibilidad. Para lograrlo, se emplea un manipulador industrial ABB IRB 140 junto con un controlador IRC5, encargados de ejecutar trayectorias precisas sobre una superficie plana para escribir los nombres de los integrantes del equipo y trazar una figura decorativa, utilizando una herramienta diseñada especialmente para esta tarea.

El proyecto requiere la aplicación de conceptos fundamentales de robótica industrial, incluyendo la calibración de herramientas, la programación en lenguaje RAPID y la generación de trayectorias. Asimismo, el sistema integra el uso de entradas y salidas digitales para lograr una coordinación sincronizada entre el robot y una banda transportadora. Todo este entorno de trabajo se modela, analiza y valida previamente mediante el software de simulación RobotStudio, lo que permite asegurar el correcto funcionamiento de los componentes en un entorno virtual antes de su ejecución física.

**Especificaciones y restricciones del sistema:**

| Parámetro | Valor |
|-----------|-------|
| Tamaño de la torta | ~20 personas |
| Rango de velocidades | 100 – 1000 (unidades RAPID) |
| Tolerancia máxima en Z | ±10 mm |
| Tipo de movimiento | Trazo continuo |
| Posición inicial/final | Home |
| Decoración requerida | Nombres + figura adicional (Pacman) |

### 1.2 Descripción de la solución

El desarrollo de la solución se ejecutó siguiendo una metodología secuencial, partiendo desde el diseño de los elementos hasta la simulación y posterior validación física:

#### a. Diseño y definición de la herramienta
El primer paso consistió en desarrollar un efector final adaptado a las medidas del flange del ABB IRB 140. La pieza fue modelada en **Autodesk Fusion 360** y fabricada mediante impresión 3D en **PETG**. Una vez lista, la geometría se exportó en formato `.SAT`, lo que permitió que RobotStudio la reconociera correctamente para acoplarla al manipulador en el entorno virtual.

#### b. Modelado del objeto de trabajo (Torta virtual)
Posteriormente, se diseñó la caja que representa la superficie de la torta. Al igual que la herramienta, este elemento se modeló en **Autodesk Fusion 360** para asegurar las proporciones correctas. Sobre este espacio virtual se planificó la distribución geométrica para el trazado de los tres nombres de los integrantes del equipo y la figura de un **Pacman**.

<div align="center">
  <img src="imagenes/Plano pastel-1.png" alt="Plano del pastel" width="600px">
  <br><em>Figura 1. Plano de distribución de trayectorias sobre el pastel</em>
</div>

#### c. Configuración de parámetros fundamentales (TCP, Wobj y Targets)
Con la herramienta y el objeto de trabajo posicionados, se procedió a configurar la base del control espacial:
- Se definió el **TCP** (Tool Center Point) virtual asociado a la punta del marcador.
- Se creó el sistema de coordenadas de la pieza o **Workobject** alineado con la caja.
- Se programaron los *Targets* (puntos de destino) y las trayectorias para asegurar que el robot trazara los caracteres y el Pacman con la interpolación adecuada.

#### d. Implementación de Smart Components
Para dotar de realismo a la simulación y emular el proceso de una línea de producción, se configuraron **Smart Components** en RobotStudio. Estos componentes inteligentes permitieron simular la cinemática de la caja desplazándose sobre la banda transportadora, reflejando el comportamiento que tendría el sistema en la vida real.

#### e. Lógica de Entradas y Salidas (E/S) y Station Logic
La coordinación entre el manipulador y los Smart Components de la banda transportadora se logró mediante la configuración de señales digitales interconectadas en el entorno de *Station Logic*. La arquitectura de control quedó definida de la siguiente manera:
- **`DI_01`**: Inicia el ciclo principal, ejecutando el código RAPID.
- **`DI_02`**: Posiciona el robot en postura de mantenimiento.
- **`DI_03`**: Activa la banda en reversa para dejar el pastel "a punto" en la posición de inicio.
- **Combinación `DI_02` + `DI_03`**: Al oprimirse simultáneamente, envían al robot a su posición *Home*.

<div align="center">
  <img src="imagenes/Logica_Estacion.png" alt="Lógica de la Estación en RobotStudio" width="800px">
  <br><em>Figura 2. Diagrama de bloques de la Station Logic interconectando entradas, controlador y Smart Components</em>
</div>

#### f. Simulación Final y Pruebas Físicas
Finalmente, se ejecutó la simulación completa comprobando que los Smart Components, las E/S y las trayectorias interactuaran sin colisiones ni errores. Una vez validado, el código fue transferido al controlador IRC5 físico, donde se realizaron ajustes menores del Workobject debido a las tolerancias de ubicación de la banda transportadora real.

---

## 2. Diagrama de flujo de acciones del robot

<div align="center">
  <img src="imagenes/Diagrama_flujo.png" alt="Diagrama de flujo" width="480px">
  <br><em>Figura 3. Diagrama de flujo del programa principal del robot</em>
</div>

El flujo de ejecución sigue la lógica de un bucle infinito (`WHILE TRUE`) que evalúa las entradas digitales y sus combinaciones:

<pre>
Inicio
 └─ WHILE TRUE
     ├─ DI_02 = 1 Y DI_03 = 1? → Enviar robot a posición Home
     ├─ DI_01 = 1? → Iniciar ciclo principal (RAPID):
     │    ├─ Avanzar banda
     │    ├─ Esperar posicionamiento
     │    ├─ Activar DO_01
     │    ├─ Ejecutar trayectorias (Letras → Pacman)
     │    ├─ Avanzar banda
     │    └─ Desactivar DO_01
     ├─ DI_02 = 1? → Enviar robot a posición de Mantenimiento
     └─ DI_03 = 1? → Mover banda en reversa (posicionar pastel a punto)
</pre>

---

## 3. Plano de planta

<div align="center">
  <img src="imagenes/Plano Montaje Lab 1-1.png" alt="Plano de planta" width="850px">
  <br><em>Figura 4. Plano de planta con la ubicación del robot, banda transportadora y zona de trabajo</em>
</div>

El plano muestra la distribución espacial de los elementos del sistema:
- **Robot ABB IRB 140**: posicionado en el centro del área de trabajo.
- **Banda transportadora**: ubicada a un lado del robot, con su eje longitudinal perpendicular al alcance frontal del robot.
- **Zona de decoración (pastel)**: área de trabajo accesible dentro del espacio de trabajo del robot.

---

## 4. Descripción de las funciones utilizadas

A continuación se describen las instrucciones de movimiento RAPID empleadas en el desarrollo de la práctica:

### 4.1 Instrucciones de movimiento

| Función | Descripción | Uso en el laboratorio |
|---------|-------------|----------------------|
| `MoveJ` | Movimiento en espacio de juntas (joint space). El robot elige la ruta más directa articulación por articulación. | Desplazamientos rápidos entre posiciones alejadas (Home, puntos previos). |
| `MoveL` | Movimiento lineal en espacio cartesiano. El TCP se desplaza en línea recta. | Trazado de segmentos rectos en las letras. |
| `MoveC` | Movimiento circular en espacio cartesiano. Requiere punto intermedio y punto final. | Trazado de curvas y arcos (letras A, C, D, E, J, N, U y contorno de la figura de Pacman). |

### 4.2 Parámetros de velocidad y zona

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `v100` | 100 mm/s | Velocidad de ejecución de todas las trayectorias (requerimiento del laboratorio). |
| `z10` | 10 mm | Zona de suavizado general. El robot no para exactamente en el punto, suaviza la trayectoria con radio 10 mm. |
| `z1` | 1 mm | Zona de suavizado reducida, usada en esquinas pronunciadas (ej: bordes de la boca del Pacman) para mayor precisión. |
| `z100` | 100 mm | Zona amplia usada en el movimiento Home para un retorno fluido. |

### 4.3 Instrucciones de E/S digitales

| Función | Descripción |
|---------|-------------|
| `SET <señal>` | Activa (pone en 1) una salida digital. |
| `RESET <señal>` | Desactiva (pone en 0) una salida digital. |
| `WaitTime <t>` | Pausa la ejecución durante `t` segundos. |
| `IF DI_XX = 1 THEN` | Evaluación condicional de una entrada digital. |

### 4.4 Señales digitales definidas

| Señal | Tipo | Función |
|-------|------|---------|
| `DI_01` | Entrada | Inicia el ciclo principal (ejecuta el código RAPID). |
| `DI_02` | Entrada | Envía el robot a posición de mantenimiento. |
| `DI_03` | Entrada | Mueve la banda en reversa para posicionar el pastel a punto. |
| `DI_02` + `DI_03` | Entradas | Pulsadas al tiempo, envían el robot a posición *Home*. |
| `DO_01` | Salida | Indicador de ejecución de rutina principal / control hacia el Smart Component. |
| `DO_02` | Salida | Indicador de modo mantenimiento. |
| `DO_03` | Salida | Control del Smart Component para el movimiento en reversa. |

### 4.5 Datos de herramienta y workobject

| Dato | Tipo | Descripción |
|------|------|-------------|
| `Marcador` | `tooldata` | Define el TCP y masa de la herramienta activa (marcador adaptado). |
| `Pastel` | `wobjdata` | Define el sistema de coordenadas del objeto de trabajo (superficie de la torta). |

---

## 5. Arquitectura y Modelado de la Herramienta

<div align="center">
  <img src="imagenes/Herramienta_Vista1.png" alt="Vista isométrica de la herramienta" width="350px">
  &nbsp;&nbsp;
  <img src="imagenes/Herramienta_Vista2.png" alt="Vista lateral de la herramienta" width="350px">
  <br><em>Figura 5. Vistas y prototipo de la herramienta de decoración diseñada en Fusion 360</em>
</div>

### 5.1 Parámetros y Fabricación

Para llevar a cabo la tarea de trazado continuo sobre el objeto de trabajo, se desarrolló un efector final a medida empleando **Autodesk Fusion 360**. La pieza se materializó utilizando técnicas de manufactura aditiva, específicamente impresa en filamento **PETG** para garantizar mayor resistencia mecánica. Los datos técnicos integrados en el controlador para establecer el Tool Center Point (TCP) son los siguientes:

* **Coordenadas Cartesianas (TCP):** X = 85.45 mm | Y = -1.13 mm | Z = 118.934 mm
* **Cuaternión de Orientación:** [0.866, 0, 0.5, 0] (Equivalente a una inclinación de aproximadamente 30° sobre el eje X)
* **Masa estimada:** 1 kg.

### 5.2 Criterios de Ingeniería

* **Interfaz de montaje y compatibilidad:** Se respetaron rigurosamente las medidas del flange mecánico documentadas en el datasheet oficial del ABB IRB 140. La pieza exportada en formato `.SAT` facilitó la integración perfecta en RobotStudio.
* **Sistema de fijación:** Incorpora una cavidad de acople con un mecanismo de sujeción a presión que estabiliza el marcador durante las rutas.
* **Aproximación angular:** La configuración a 30 grados frente al eje vertical resultó ser un factor clave de diseño para mejorar el alcance (reachability), evitar singularidades y prevenir colisiones entre la muñeca del robot y la superficie horizontal de la torta.

---

## 6. Lógica y Programación en RAPID

Toda la secuencia de trayectorias y el control de E/S se estructuraron bajo un enfoque de programación modular. El script principal puede consultarse en su archivo respectivo:

👉 **[Consultar Código_RAPID.md](./Código_RAPID.md)**

### 6.1 Arquitectura del Software

El código fuente está fragmentado en submódulos especializados para optimizar su lectura y facilitar la corrección de puntos:

* **Variables de Entorno:** Declaración global de las herramientas (`tooldata`) y sistemas de referencia (`wobjdata`), junto con todos los `robtargets`.
* **Núcleo de Ejecución:** El procedimiento `PROC main()` aloja la máquina de estados y las condiciones de evaluación (DI/DO) para interactuar con la banda transportadora y gestionar el ciclo, home y mantenimiento.
* **Desplazamientos de Seguridad:** Subrutinas `Path_Home()` para la postura neutral y `Path_Mantenimiento()` para la posición segura de inspección.
* **Trazado de Caracteres:** Se implementó una subrutina dedicada a cada letra específica que conforma los nombres requeridos (por ejemplo, `Path_A_1()`, `Path_J()`, `Path_N_1()`, etc.).
* **Elemento Gráfico:** La rutina `Path_10()` agrupa el movimiento complejo encargado de dibujar el Pacman.

### 6.2 Criterios de Movimiento

* **Dinámica Unificada:** Se garantizó una velocidad constante de `v100` (100 mm/s) durante el contacto con la torta para mantener el flujo del marcador.
* **Interpolación de Zonas:** Se implementó un radio de empalme `z10` en transiciones largas para dar fluidez al brazo, pero se forzó una zona estricta de `z1` en esquinas críticas (como los vértices de las letras o los ángulos agudos de la boca del Pacman) para no redondear la geometría esperada.

---

## 7. Evidencias Multimedia

En este apartado se compilan los registros visuales que certifican el desempeño de las trayectorias programadas.

### 7.1 Validación Virtual
> 🎥 *[Espacio reservado para anexar el video del gemelo digital en RobotStudio]*

### 7.2 Validación Física
> 🎥 *[Espacio reservado para anexar la ejecución real con el hardware en el laboratorio]*

---

<div align="center">
  <sub>← <a href="../README.md">Volver al repositorio principal</a> | <a href="./Código_RAPID.md">Ver código RAPID completo →</a></sub>
</div>
