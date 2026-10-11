---
title: Práctica 2 — Integración de Objetos Inteligentes con TCP/IP
status: approved
created: 2026-10-08
updated: 2026-10-08
sources:
  - "../../../../Práctica_2.md"
companions:
  - "../../architecture/architecture-Practica2-2026-10-08/ARCHITECTURE-SPINE.md"
  - "../../EXECUTION-PLAN-DEFENSE.md"
  - "../../protocol/TCP-APPLICATION-PROTOCOL.md"
  - "../../REQUIREMENTS-ACCEPTANCE-TESTS.md"
  - "addendum.md"
---

# PRD: Integración de Objetos Inteligentes con TCP/IP

## 0. Propósito y autoridad

Este PRD guía al grupo, los implementadores y la elaboración del informe técnico de la Práctica 2 de SIS-234. `Práctica_2.md` es la fuente oficial para requerimientos, entregables y evaluación. La arquitectura PlatformIO, la variante de tres LEDs, los límites de distancia, el protocolo IoT-Range/1 y las metas cuantitativas que no estén fijados por la práctica se identifican como decisiones del proyecto, no como mandatos del docente. Los cuatro artefactos técnicos compañeros contienen estructura, protocolo, plan de defensa y matriz de verificación detallada.

## 1. Visión

Construir y demostrar un sistema distribuido pequeño con dos objetos inteligentes ESP32 y un servidor TCP ejecutado en una computadora. El objeto sensor mide una distancia ultrasónica y la reporta por WiFi/TCP. El servidor interpreta la medida y envía una orden al objeto actuador, que representa tres rangos con tres LEDs.

El trabajo debe resultar demostrable en vivo y defendible individualmente. El informe técnico y las evidencias deben permitir entender la arquitectura, el circuito, las clases, el protocolo, el plan de validación y los resultados que realmente se midan.

## 2. Usuarios y contexto

### 2.1 Trabajo que se busca realizar

Un grupo académico de 2 o 3 integrantes diseña, implementa, prueba y presenta un prototipo que demuestra sensores IP sobre IEEE 802.11, TCP/IP, arquitectura cliente-servidor y un protocolo de aplicación propio.

### 2.2 Participantes

- **Integrantes del grupo:** diseñan, construyen, documentan y deben poder explicar el sistema; cada persona debe dejar commits desde su propia cuenta GitHub.
- **Docente/evaluador:** revisa el repositorio e informe, observa la demo y realiza una defensa individual.

El número de integrantes, nombres y cuentas no constan en el material disponible y quedan abiertos para organizar el trabajo y la evidencia individual.

### 2.3 Recorrido de uso principal

Durante la demo, un integrante inicia el servidor en la computadora y energiza ambos ESP32. Cada placa abre una sesión TCP y se registra. Al variar la distancia frente al HC-SR04, el servidor clasifica cada lectura y el actuador cambia sus LEDs; el grupo muestra los rangos y límites, explica los mensajes observados y puede demostrar la reacción segura ante un error.

### 2.4 Aprendizajes evaluados por la práctica (§2)

- **Conceptuales:** redes de sensores IP sobre IEEE 802.11, modelo TCP/IP, cliente-servidor y transporte TCP.
- **Procedimentales:** configuración de conectividad IP/WiFi, comunicación cliente-servidor con sockets y diseño/documentación de un protocolo simple de aplicación.
- **Actitudinales:** colaboración, responsabilidad individual, orden y disciplina durante análisis, diseño y construcción. Se reflejan en commits de cada integrante y evidencia clara de tareas/aporte.

## 3. Glosario

- **Objeto sensor:** ESP32 con HC-SR04, cliente TCP que produce medidas.
- **Objeto actuador:** ESP32 con tres LEDs y resistencias limitadoras, cliente TCP que aplica órdenes `SET`.
- **Servidor:** proceso Python en una PC que acepta ambos clientes, valida mensajes, clasifica medidas y remite comandos.
- **IoT-Range/1:** protocolo de aplicación de texto sobre TCP que se especifica en el companion correspondiente.
- **Rango de trabajo:** intervalo físico de distancias para el cual el grupo declara y verifica la exactitud del sensor.
- **Estado seguro:** rojo, amarillo y verde apagados.
- **Medida inválida:** lectura no finita, no positiva, timeout del sensor o medida declarada fuera del rango físico validado.

## 4. Alcance y restricciones

### 4.1 En alcance

- Un objeto sensor ultrasónico y un objeto actuador con mínimo tres LEDs, además de una PC que ejecuta el servidor TCP.
- Ambos ESP32 actúan como clientes independientes e inician conexiones al servidor sobre la misma red WiFi IEEE 802.11.
- Clasificación centralizada en Python y protocolo de aplicación propio sobre sockets TCP.
- Un único proyecto PlatformIO con entornos `sensor` y `actuator`, código fuente separado por filtros y `src/shared/` solo para clases realmente comunes.
- Informe Markdown, repositorio GitHub documentado, anexos y pruebas planeadas/ejecutadas con resultados honestos.

### 4.2 Restricciones

- Firmware ESP32 en C++ con framework Arduino; placas ESP32 DOIT DEVKIT V1.
- Pinout aprobado: sensor TRIG GPIO19 y ECHO GPIO18 con adaptación de 5 V a 3.3 V; actuador LED rojo GPIO4, amarillo GPIO21 y verde GPIO22. Cada LED usa resistencia serie de 220 o 330 Ω.
- Servidor Python 3 usando sockets TCP.
- La computadora servidor y los ESP32 comparten una red local WiFi.
- No hay comunicación directa ESP32 a ESP32; el servidor es intermediario obligatorio.
- La selección de tres LEDs y reglas exactas de salida son decisiones aprobadas del proyecto. La práctica permite también un servomotor, pero esa alternativa no se usa.
- No agregar nube, base de datos, frontend, aplicación móvil, MQTT, HTTP ni funciones ajenas a la práctica.
- Los circuitos y pines se documentarán según el hardware y montaje que se validen; no se asumirán pines sin decisión y verificación.

### 4.3 Separación de lo oficial y lo decidido por el grupo

| Tema | Obligación oficial | Decisión del proyecto |
| --- | --- | --- |
| Sistema | Sensor ultrasónico cliente, actuador cliente (mínimo 3 LEDs o servo) y servidor TCP en PC; RF1–RF4 (§3, §3.1). | Dos ESP32 DOIT DEVKIT V1 y tres LEDs; el servidor intermedia obligatoriamente. |
| Rangos | Al menos tres rangos contiguos y sin solapamiento; el grupo define lógica (§3.1). | `0<d<20`, `20≤d<40`, `d≥40` cm; inválido/error produce LEDs apagados. |
| Protocolo | Protocolo de aplicación propio definido y documentado (§3.1, §4.2). | IoT-Range/1 con `REGISTER`, `POST`, `SET`, `ERROR` y respuestas `ACK`; heartbeat actuador propuesto. |
| NFR | Declarar y verificar estabilidad, exactitud, respuesta extremo a extremo y frecuencia con valores medibles (§3.2). | Objetivos iniciales: ≥10 min, error ≤±3 cm, respuesta ≤1 s y muestreo ≥2/s. No son resultados. |
| Desarrollo | Código legible, modular/POO, documentado y conforme a convenciones (§3.2). | Un solo PlatformIO con dos entornos; Python fuera de `src/`. |

## 5. Funcionalidades y requisitos

Los RF-01 a RF-04 son el mapeo trazable de los RF1–RF4 oficiales. RF-05 en adelante añade comportamiento/protocolo aprobado o propuesto por el grupo para que la solución sea verificable; no se atribuye a la rúbrica como requisito textual.

### 5.1 Medición y clasificación

#### RF-01 — Medir distancia (oficial RF1, §3.1)

El objeto sensor mide la distancia entre el HC-SR04 y el objeto dentro del rango de trabajo declarado.

**Aceptación:** el procedimiento obtiene lecturas y distingue una lectura válida de timeout/lectura inválida. La frecuencia se verifica según NFR-04.

#### RF-02 — Clasificar en rangos (oficial RF2, §3.1)

El servidor interpreta una medida válida y la clasifica en al menos tres rangos contiguos y sin solapamiento.

**Aceptación:** aplica exactamente las fronteras del proyecto: `0<d<20 cm`, `20≤d<40 cm`, `d≥40 cm`; pruebas en 19.9, 20, 20.1, 39.9, 40 y 40.1 cm.

#### RF-03 — Actuar según el rango (oficial RF3, §3.1)

El objeto actuador muestra comportamiento distinto para cada rango. Los LEDs ajenos al rango deben permanecer apagados.

**Aceptación:** rango bajo: rojo parpadea a 4 Hz; amarillo y verde apagados. Rango medio: amarillo fijo; rojo y verde apagados. Rango alto: verde fijo; rojo y amarillo apagados. El parpadeo usa cuatro ciclos completos por segundo y no bloquea la recepción TCP.

### 5.2 Comunicación y seguridad de control

#### RF-04 — Integración por TCP/WiFi (oficial RF4, §3.1)

El sensor envía sus lecturas al servidor y el servidor envía comandos al actuador por sockets TCP sobre WiFi IEEE 802.11 utilizando el protocolo de aplicación del grupo.

**Aceptación:** se evidencia que los dos ESP32 inician conexiones separadas al servidor, que las medidas recorren sensor→servidor y que los comandos recorren servidor→actuador; no existe enlace directo entre placas.

#### RF-05 — Registrar roles (decisión de protocolo del grupo)

Cada cliente se registra como `sensor` o `actuator` al establecer una sesión TCP nueva.

**Aceptación:** el servidor acepta un único cliente de cada rol por vez, responde al registro válido y rechaza rol desconocido o segunda sesión simultánea del mismo rol. Reconexion requiere nuevo registro.

#### RF-06 — Aplicar y validar IoT-Range/1 (decisión de protocolo del grupo)

Los participantes intercambian mensajes delimitados y validados según el companion IoT-Range/1.

**Aceptación:** parser reconstruye líneas TCP fragmentadas y varias líneas recibidas juntas; valida campos, rol, secuencia, límites y valores LED; mensajes inválidos se reportan y nunca provocan un estado activo no válido.

#### RF-07 — Mantener estado seguro ante fallas (decisión del grupo)

Una medida inválida, timeout, orden inválida, dato obsoleto o pérdida de enlace deja los tres LEDs apagados.

**Aceptación:** se verifican errores del HC-SR04, inactividad del sensor, desconexión de cada ESP32 y pérdida del servidor. Al reconectar no se reutilizan medidas viejas para encender LEDs.

#### RF-08 — Recuperar la comunicación (decisión del grupo)

Sensor y actuador pueden reconectarse de forma independiente al servidor y registrarse de nuevo.

**Aceptación:** se documenta cada desconexión, reintento, registro y recuperación; el estado inicial tras reconexión es seguro hasta obtener una lectura nueva válida.

## 6. Requisitos no funcionales

La práctica (§3.2) requiere declarar y verificar cada atributo, pero presenta sus cifras como referencias. El grupo adopta las metas de la tabla como objetivos de aceptación. No se consideran cumplidas hasta medirlas en el prototipo.

| ID | Atributo | Objetivo del proyecto | Método de aceptación |
| --- | --- | --- | --- |
| NFR-01 | Estabilidad/confiabilidad | Operación continua ≥10 minutos sin reinicios ni bloqueos no esperados. | Registrar duración, reinicios, bloqueos y desconexiones; separar fallas intencionales de espontáneas. |
| NFR-02 | Exactitud | Error absoluto máximo ≤3 cm respecto de cinta métrica en el rango inicial 5–100 cm. Cinco lecturas por distancia de referencia. | Medir puntos de referencia, comparar cada lectura, reportar error absoluto por punto, máximo observado, condiciones y rango realmente alcanzado. |
| NFR-03 | Respuesta | Máximo 1 segundo desde lectura/cambio de rango hasta actuación correspondiente. | Marcas de tiempo extremo a extremo para todas las pruebas seleccionadas; informar máximo observado, no solo promedio. |
| NFR-04 | Frecuencia de muestreo | ≥2 lecturas válidas por segundo durante prueba con objeto dentro del rango utilizable. | Contabilizar lecturas válidas enviadas en un intervalo conocido; reportar por separado intentos, lecturas válidas, inválidas y pérdidas. El objetivo se evalúa en condiciones de sensor que permitan eco válido. |
| NFR-05 | Calidad del código | Legible, modular/POO, documentado y conforme a convenciones de C++/Python. | Revisar clases/responsabilidades, estructura, nombres, documentación y cobertura del código frente al PRD. |
| CR-D1 | Presentación física de Demo (criterio de rúbrica, no NFR §3.2) | Montaje ordenado/seguro y operación integral vía servidor TCP/WiFi. | Lista de revisión visual y demostración de comunicación, rangos y límites, incluidos estímulos que solicite el docente. |

## 7. Entregables y éxito de evaluación

### 7.1 Entregables oficiales (§4, §6)

1. Prototipo completamente operativo.
2. Informe técnico Markdown en el repositorio con las ocho secciones oficiales: requerimientos; análisis/diseño; desarrollo/implementación; pruebas/validaciones; resultados; conclusiones; recomendaciones; anexos.
3. Diagramas de arquitectura/despliegue, circuito de cada objeto inteligente, clases estructural y secuencia con mensajes; protocolo documentado.
4. Descripción de módulos/clases y fragmentos clave con enlaces a fuentes, sin copiar el código completo en el informe.
5. Repositorio GitHub accesible al docente con fuente documentada, informe y anexos pertinentes (diagramas, esquemas, fotos y evidencias de pruebas).
6. Commits de cada integrante desde su propia cuenta como evidencia de contribución individual.
7. Enlace del repositorio entregado por LMS-UCB hasta el martes 13 de octubre de 2026; basta que lo envíe un integrante.

### 7.2 Medición de éxito frente a rúbrica (§5)

| Componente | Peso | Señal de nivel 4 |
| --- | ---: | --- |
| Informe técnico | 1/3; cuatro criterios de 0–4, total 16; porcentaje = puntos/16×100 | Requisitos medibles y trazables; cuatro clases de diagramas exigidos y protocolo completo; implementación POO documentada que cubra lo declarado; pruebas completas para RF/NFR con análisis de errores; resultados cuantificables, crítica, conclusiones, recomendaciones y anexos. |
| Demo | 1/3; nivel global 0–4 | Prototipo completo/estable, datos vía servidor TCP sobre WiFi, respuesta correcta en rangos y límites requeridos en vivo, montaje ordenado/seguro. |
| Defensa individual | 1/3; nivel global 0–4 | Respuestas profundas; cada estudiante explica código y circuitos y justifica decisiones; aporte individual visible en commits propios. |

La nota final es el promedio de los tres componentes. Un componente no presentado o sin evidencia puede recibir nivel 0.

### 7.3 Señales de éxito del proyecto

- **SM-01:** flujo completo sensor→servidor→actuador demostrable por TCP/WiFi en dos clientes independientes; valida RF-04 y PR-01.
- **SM-02:** entradas de los tres rangos producen las tres salidas exactas y los límites se prueban; valida RF-02/RF-03.
- **SM-03:** se miden NFR-01–NFR-04 contra los objetivos declarados y se informa máximo/tasa/error, sin resultados inventados.
- **SM-04:** informe, fuentes, diagramas y anexos cumplen los entregables; cada miembro demuestra commit propio; valida el criterio de entrega y la defensa.
- **Contramétrica de seguridad:** ninguna lectura inválida/antigua debe aumentar la distancia aparente ni activar el verde; los tres LEDs apagados siguen siendo el estado seguro ante fallo.

## 8. Restricciones y no objetivos

- No se implementa comunicación directa entre ESP32 ni se mueve la clasificación fuera del servidor.
- No se sustituye TCP por HTTP, MQTT o WebSockets.
- No se añaden nube, base de datos, frontend, aplicación móvil ni funciones no exigidas.
- No se presentan objetivos como mediciones logradas, ni resultados, fotos o pruebas físicas ficticias.
- El funcionamiento vivo se evalúa en Demo (§5.3), separado de la puntuación del informe; el informe debe documentar el plan/resultados de pruebas, pero la demostración no reemplaza el informe (§5.2.1).

## 9. Decisiones aprobadas y validaciones pendientes

1. **Validación física:** usar rango inicial 5–100 cm y cinco lecturas por distancia de referencia; reportar error máximo absoluto frente a cinta métrica. El objetivo ≤3 cm aún debe comprobarse con el módulo real.
2. **Pruebas exactas de límites:** aplicar los seis valores decimales al clasificador automatizado. En físico registrar distancia de referencia, lectura real recibida y salida según esa lectura; no asumir separación física de 0.1 cm.
3. **Pines/circuito aprobados:** TRIG GPIO19, ECHO GPIO18 con adaptación 5→3.3 V, rojo GPIO4, amarillo GPIO21 y verde GPIO22; una resistencia serie de 220/330 Ω por LED. Verificar el módulo HC-SR04, polaridad y comportamiento de apagado durante reset en el montaje real antes de energizar.
4. **Grupo y commits:** confirmar integrantes (2 o 3), cuentas y reparto de tareas para evidenciar contribución individual.
5. **Validación de IoT-Range/1:** heartbeat inicial cada 2 s, timeout 6 s; host servidor configurable, puerto 5000, línea LF/CRLF. Parámetros sujetos a validación real.
6. **Conexión local:** acordar cómo obtener/reservar la IP del PC en la WiFi de demostración.
7. **Hardware exacto:** validar el modelo/módulo HC-SR04 disponible, alimentación, umbral de TRIG, adaptación ECHO y revisión de la placa DOIT V1.

## 10. Supuestos indexados

- Los objetivos NFR del §6 son metas elegidas por el grupo a partir de las cifras de referencia del enunciado, no umbrales textuales obligatorios ni resultados.
- La política de tres LEDs y umbrales 20/40 cm viene del brief aprobado por el usuario; los ejemplos de rangos de §3.1 no los reemplazan.
- Límites de protocolo, heartbeat, puerto predeterminado y tiempos de reconexión son decisiones técnicas propuestas que se revisarán antes de implementar.
