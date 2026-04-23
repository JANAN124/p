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
