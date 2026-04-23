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
<h4>Estudiantes: Juan Diego Sáenz Ardila · Alejandra Sofia Monroy Socha</h4>

<p>
  <img alt="ABB IRB140" src="https://img.shields.io/badge/Robot-ABB%20IRB%20140-CC0000?logoColor=white">
  <img alt="RobotStudio" src="https://img.shields.io/badge/Software-RobotStudio-41C9D0?logoColor=white">
  <img alt="RAPID" src="https://img.shields.io/badge/Lenguaje-RAPID-FF6600?logoColor=white">
  <img alt="Estado" src="https://img.shields.io/badge/Estado-Completo-brightgreen">
</p>
</div>

---

## Tabla de contenido

1. [Introducción y descripción de la solución](#1-introducción-y-descripción-de-la-solución)
2. [Diagrama de flujo de acciones del robot](#2-diagrama-de-flujo-de-acciones-del-robot)
3. [Plano de planta](#3-plano-de-planta)
4. [Descripción de las funciones utilizadas](#4-descripción-de-las-funciones-utilizadas)
5. [Diseño de la herramienta](#5-diseño-de-la-herramienta)
6. [Código RAPID](#6-código-rapid)
7. [Videos](#7-videos)

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
| Decoración requerida | Nombres + figura adicional (corazón) |

### 1.2 Descripción de la solución

La solución se desarrolló de forma progresiva: primero en el entorno virtual (RobotStudio) y luego en el físico.

#### a. Definición de la herramienta

Se consultó el datasheet del robot ABB IRB 140 para identificar las especificaciones del flange. Con base en esto, se realizó el modelado CAD en **Autodesk Inventor**, incorporando consideraciones de manufactura aditiva (impresión 3D).

#### b. Caracterización del entorno de trabajo

Se tomaron mediciones en laboratorio para definir un sistema coherente entre el entorno real y la simulación:
- Posición y dimensiones de la banda transportadora.
- Distancia entre el robot y la zona de trabajo.

#### c. Modelado del objeto de trabajo

Con la zona de trabajo definida, se modeló el pastel considerando la distribución espacial de los nombres y la figura decorativa:

<div align="center">
  <img src="imagenes/Plano pastel-1.png" alt="Plano del pastel" width="600px">
  <br><em>Figura 1. Plano de distribución de trayectorias sobre el pastel</em>
</div>

#### d. Simulación y configuración en RobotStudio

En el entorno de simulación se realizó:
- Carga del robot y la herramienta diseñada.
- Definición del **TCP** (Tool Center Point).
- Creación del workobject virtual.
- Programación de trayectorias: posición *home*, escritura de nombres, figura decorativa (corazón) y retorno a *home*.

#### e. Integración del sistema

Se integraron los distintos componentes del sistema para coordinar el robot con la banda transportadora mediante señales digitales (DI/DO). La sincronización se realizó por parámetros temporales (no por velocidad de banda), definiendo:
- Tiempos de activación de la banda.
- Tiempos de espera entre llegada de pasteles.
- Tiempos de espera asociados al robot.

Las entradas digitales físicas se implementaron como botones y las salidas como indicadores luminosos.

#### f. Implementación y pruebas en laboratorio

Tras validar el sistema en simulación, se implementó físicamente. Aspectos clave:

- **TCP**: La calibración experimental presentó un error de ~6 mm, por lo que se utilizó el TCP virtual (error nulo).
- **Workobject**: Se usó el workobject real, pues la posición de la banda real no coincidía exactamente con el modelo.
- Se ajustaron experimentalmente los tiempos de activación de banda, tiempos de espera entre ciclos y sistemas coordenados del objeto de trabajo.

---

## 2. Diagrama de flujo de acciones del robot

<div align="center">
  <img src="imagenes/Diagrama_flujo.png" alt="Diagrama de flujo" width="480px">
  <br><em>Figura 2. Diagrama de flujo del programa principal del robot</em>
</div>

El flujo de ejecución sigue la lógica de un bucle infinito (`WHILE TRUE`) que evalúa las entradas digitales:

```
Inicio
 └─ WHILE TRUE
     ├─ DI_03 = 1? → Activar banda en reversa (BWD_Conveyor, 4 s)
     ├─ DI_01 = 1? → Secuencia completa:
     │    ├─ Avanzar banda (FWD_Conveyor, 4 s)
     │    ├─ Esperar posicionamiento (5 s)
     │    ├─ Activar DO_01
     │    ├─ Ejecutar trayectorias (Home → Letras → Corazón → Home)
     │    ├─ Avanzar banda (4 s)
     │    └─ Desactivar DO_01
     └─ DI_02 = 1? → Ir a posición de mantenimiento → Home
```

---

## 3. Plano de planta

<div align="center">
  <img src="imagenes/Plano Montaje Lab 1-1.png" alt="Plano de planta" width="850px">
  <br><em>Figura 3. Plano de planta con la ubicación del robot, banda transportadora y zona de trabajo</em>
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
| `MoveC` | Movimiento circular en espacio cartesiano. Requiere punto intermedio y punto final. | Trazado de curvas y arcos (letras A, C, D, E, J, N, U y figura del corazón). |

### 4.2 Parámetros de velocidad y zona

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `v100` | 100 mm/s | Velocidad de ejecución de todas las trayectorias (requerimiento del laboratorio). |
| `z10` | 10 mm | Zona de suavizado general. El robot no para exactamente en el punto, suaviza la trayectoria con radio 10 mm. |
| `z1` | 1 mm | Zona de suavizado reducida, usada en esquinas pronunciadas (ej: punta del corazón) para mayor precisión. |
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
| `DI_01` | Entrada | Activa la secuencia completa de decoración. |
| `DI_02` | Entrada | Envía el robot a posición de mantenimiento. |
| `DI_03` | Entrada | Activa el control manual de la banda (reversa). |
| `DO_01` | Salida | Indicador de ejecución de rutina principal. |
| `DO_02` | Salida | Indicador de modo mantenimiento. |
| `DO_03` | Salida | Indicador de activación de banda en reversa. |
| `FWD_Conveyor` | Salida | Activa la banda en dirección hacia adelante. |
| `BWD_Conveyor` | Salida | Activa la banda en dirección hacia atrás. |

### 4.5 Datos de herramienta y workobject

| Dato | Tipo | Descripción |
|------|------|-------------|
| `Marcador` | `tooldata` | Define el TCP y masa de la herramienta activa (marcador adaptado). |
| `Pastel` | `wobjdata` | Define el sistema de coordenadas del objeto de trabajo (superficie de la torta). |

---

## 5. Diseño de la herramienta

<div align="center">
  <img src="imagenes/Ensamble herramienta-1.png" alt="Diseño de la herramienta" width="850px">
  <br><em>Figura 4. Ensamble y diseño detallado de la herramienta de decoración</em>
</div>

### 5.1 Especificaciones de diseño

La herramienta fue diseñada en **Autodesk Inventor** y fabricada mediante **manufactura aditiva (impresión 3D)**. Las características principales son:

| Parámetro | Valor |
|-----------|-------|
| TCP X | 85.45 mm |
| TCP Y | -1.13 mm |
| TCP Z | 118.934 mm |
| Orientación (quaternion) | [0.866, 0, 0.5, 0] (~30° sobre eje X) |
| Masa estimada | 1 kg |

### 5.2 Consideraciones de diseño

- **Acople al flange**: diseñado según especificaciones del datasheet del ABB IRB 140.
- **Sujeción del marcador**: alojamiento cilíndrico con mecanismo de presión ajustable.
- **Ángulo de inclinación**: 30° respecto al eje Z del flange, para mejorar la accesibilidad a la superficie horizontal de la torta.
- **Fabricación**: PLA mediante impresión 3D FDM.

---

## 6. Código RAPID

El programa RAPID completo puede consultarse en el archivo dedicado:

📄 **[Ver Código_RAPID.md](./Código_RAPID.md)**

### 6.1 Estructura general del módulo

```
MODULE Module1
 ├── Definición de tooldata (Marcador)
 ├── Definición de wobjdata (Pastel)
 ├── Declaración de robtargets (puntos de trayectoria)
 ├── PROC main()        ← Bucle principal con lógica DI/DO
 ├── PROC Path_Home()   ← Posición de reposo
 ├── PROC Path_Mantenimiento() ← Posición de cambio de herramienta
 ├── PROC Path_A_1()    ← Letra A (nombre Alejandra)
 ├── PROC Path_L()      ← Letra L
 ├── PROC Path_E()      ← Letra E
 ├── PROC Path_JA()     ← Letra J (nombre Juan)
 ├── PROC Path_A_2()    ← Letra A
 ├── PROC Path_N_1()    ← Letra N
 ├── PROC Path_D()      ← Letra D
 ├── PROC Path_R()      ← Letra R (nombre ARDILA)
 ├── PROC Path_A_3()    ← Letra A
 ├── PROC Path_S()      ← Letra S (nombre SAENZ)
 ├── PROC Path_J()      ← Letra J
 ├── PROC Path_U()      ← Letra U
 ├── PROC Path_A_4()    ← Letra A
 ├── PROC Path_N_2()    ← Letra N
 └── PROC Path_10()     ← Figura decorativa (corazón)
ENDMODULE
```

### 6.2 Parámetros clave

- **Velocidad**: `v100` (100 mm/s) en todas las trayectorias.
- **Zona general**: `z10` para suavizado estándar.
- **Zona especial**: `z1` en vértices pronunciados (punta del corazón, esquinas de letras) para garantizar precisión geométrica.

---

## 7. Videos

### 7.1 Simulación en RobotStudio

> 🎬 *Video de simulación — próximamente disponible*

### 7.2 Implementación con robots reales

> 🎬 *Video de implementación en laboratorio — próximamente disponible*

---

<div align="center">
  <sub>← <a href="../README.md">Volver al repositorio principal</a> | <a href="./Código_RAPID.md">Ver código RAPID completo →</a></sub>
</div>
