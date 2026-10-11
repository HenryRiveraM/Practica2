---
status: final
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - "../../Práctica_2.md"
  - "REQUIREMENTS-ACCEPTANCE-TESTS.md"
  - "EXECUTION-PLAN-DEFENSE.md"
  - "architecture/architecture-Practica2-2026-10-08/ARCHITECTURE-SPINE.md"
  - "architecture/architecture-Practica2-2026-10-08/DETAILED-ARCHITECTURE.md"
  - "protocol/TCP-APPLICATION-PROTOCOL.md"
  - "prds/prd-Practica2-2026-10-08/prd.md"
  - "prds/prd-Practica2-2026-10-08/addendum.md"
---

# Práctica2 - Desglose de épicas

## Overview

Inventario y dos épicas con 24 historias aprobados por el usuario y validados para desarrollo. No se ha iniciado implementación.

## Requirements Inventory

### Functional Requirements

**Requisitos oficiales de `Práctica_2.md`:**

- RF1: El objeto sensor mide la distancia entre el sensor ultrasónico y un objeto.
- RF2: El sistema interpreta la distancia en al menos tres rangos contiguos y sin solapamiento.
- RF3: El objeto actuador se comporta de manera diferente para cada rango detectado.
- RF4: El objeto sensor envía lecturas al servidor; el servidor envía comandos al actuador mediante sockets TCP sobre WiFi IEEE 802.11 y un protocolo de aplicación propio.

**Requisitos acordados del proyecto (no atribuidos como texto oficial del docente):**

- RF-05 (PRD): El servidor registra roles `sensor` y `actuator` en conexiones TCP independientes.
- RF-06 (PRD): El sistema valida e intercambia mensajes mediante IoT-Range/1.
- RF-07 (PRD): Ante error, medida inválida/obsoleta o pérdida de enlace, los tres LEDs quedan apagados; no se reutilizan mediciones antiguas.
- RF-08 (PRD): Los clientes pueden reconectarse de manera independiente y registrarse de nuevo.

### NonFunctional Requirements

- NFR1 / NFR-01 (meta del proyecto): operación continua mínima de 10 minutos; registrar reinicios, bloqueos y pérdidas. No se presume cumplimiento.
- NFR2 / NFR-02 (meta del proyecto): rango de evaluación inicial de 5–100 cm, cinco lecturas por distancia de referencia y error absoluto máximo objetivo de 3 cm respecto a cinta métrica.
- NFR3 / NFR-03 (meta del proyecto): respuesta extremo a extremo máxima de 1 segundo desde lectura/cambio de rango hasta salida.
- NFR4 / NFR-04 (meta del proyecto): al menos dos lecturas válidas por segundo en condiciones con eco válido; reportar aparte intentos, lecturas inválidas y pérdidas.
- NFR5 / NFR-05 (exigencia de calidad del enunciado): código legible, modular, orientado a objetos, documentado y con convenciones adecuadas para C++ y Python.
- El proyecto debe demostrar conectividad por TCP/WiFi con dos clientes independientes, y el informe y pruebas deben cubrir requisitos funcionales y metas NFR con resultados honestos.
- Los objetivos anteriores son metas sujetas a prueba; no son resultados experimentales.

### Additional Requirements

- Dos placas ESP32 DOIT DEVKIT V1; una conecta HC-SR04 y otra tres LEDs rojo/amarillo/verde.
- Resistencias limitadoras de 220 o 330 Ω, una por LED; ECHO debe tener adaptación de nivel antes de entrar al ESP32; verificar alimentación, GND común y el módulo concreto antes de energizar.
- Pinout aprobado: TRIG GPIO19, ECHO GPIO18 con adaptación 5→3.3 V; LED rojo GPIO4, amarillo GPIO21 y verde GPIO22. Una resistencia serie 220/330 Ω por LED. GPIO2 y GPIO15 se excluyen por función strapping.
- Cliente sensor emite `POST` por medida válida y `ERROR` ante lectura inválida; servidor clasifica todos los rangos.
- IoT-Range/1 usa ASCII, mensajes delimitados por LF (acepta CRLF), `REGISTER`, `POST`, `SET`, `ERROR`, `ACK`, `PING`, `PONG`, validación estricta, secuencias y límites según el documento de protocolo.
- Muestreo válido objetivo ≥2 Hz; deadman de sensor de 1.5 s desde el último `POST` válido (desde `REGISTER` si aún no hay POST). Solo un POST válido nuevo reinicia su referencia. `ERROR` invalida inmediatamente la medición anterior pero no reinicia ni extiende el deadman. Heartbeat actuador 2 s, timeout 6 s y ACK inicial de 500 ms son parámetros técnicos sujetos a validación conjunta con respuesta ≤1 s.
- El servidor atiende ambas sesiones concurrentes con un coordinador único y serializa los `SET`; al registrar/reconectar un actuador, no reutiliza `SET` ni `POST` previos.
- Estado seguro incluye boot/reset: los tres LEDs deben permanecer apagados desde energización hasta `SET` derivado de dato fresco; validar eléctricamente para GPIO en alta impedancia.
- Plan de pruebas debe separar pruebas automatizadas exactas de fronteras (19.9/20/20.1/39.9/40/40.1 cm) y pruebas físicas basadas en lecturas reales; no asumir resolución física de 0.1 cm.
- La rúbrica exige informe Markdown con ocho secciones, diagrama de arquitectura/despliegue, circuito de cada objeto, diagrama de clases y secuencia, explicación de módulos y clases, pruebas con errores/análisis, resultados cuantificables, conclusiones, recomendaciones y anexos.
- La entrega requiere fuente e informe/anexos en GitHub accesible al docente, commits de cada integrante desde su cuenta y envío de enlace por LMS-UCB hasta 13-10-2026.
- No agregar nube, base de datos, frontend, app móvil ni protocolos ajenos a TCP.
- El parpadeo rojo debe ser no bloqueante; heartbeat del actuador cada 2 s, timeout de 6 s, deadman de sensor 1.5 s y ACK de comando 500 ms son parámetros iniciales del protocolo sujetos a validación.
- Puerto inicial del servidor TCP: 5000; host/IP configurable. El servidor Python queda fuera de `src/`.
- Un proyecto PlatformIO con entornos `sensor` y `actuator`, filtros de fuentes independientes y carga individual; `src/shared/` solo contendrá código usado realmente por ambos.

### UX Design Requirements

No hay interfaz de usuario ni artefacto UX visual en el alcance; no aplica.

### FR Coverage Map

- RF1: Epic 1 — adquisición de lecturas ultrasónicas.
- RF2: Epic 2 — clasificación en tres rangos.
- RF3: Epic 2 — representación en los tres LEDs.
- RF4: Epic 1 y Epic 2 — transporte sensor→servidor y cadena completa servidor→actuador por TCP/WiFi.
- RF-05: Epic 1 y Epic 2 — registro de ambos roles.
- RF-06: Epic 1 y Epic 2 — validación de IoT-Range/1 en las interfaces utilizadas.
- RF-07: Epic 1 y Epic 2 — errores de lectura y comportamiento seguro del sistema integrado.
- RF-08: Epic 1 y Epic 2 — reconexión y nuevo registro, sin reutilizar estado vencido.

## Épicas aprobadas

### Epic 1: Recibir mediciones de distancia válidas en el servidor

El grupo puede obtener medidas ultrasónicas del objeto, enviarlas mediante una sesión IoT-Range/1 registrada por TCP/WiFi y observar en el servidor los datos válidos y los errores, con reconexión independiente del sensor.

**FRs covered:** RF1, RF4 (tramo sensor→servidor), RF-05, RF-06, RF-07 (fallos de medición), RF-08.

**Notas técnicas:** incluye la base del único proyecto PlatformIO, configuración del entorno `sensor`, conexión WiFi/TCP, `REGISTER`/`POST`/`ERROR`, configuración de host/puerto y servidor Python mínimo que procesa el rol sensor. Pruebas automatizadas de parser/framing y pruebas físicas del HC-SR04. Cumple el objetivo de rango inicial 5–100 cm, cinco lecturas por distancia de referencia y error máximo objetivo ≤3 cm sin declarar resultados anticipados.

### Story 1.1: Configurar los dos entornos PlatformIO independientes

Como integrante del grupo,
Quiero compilar por separado los firmwares del sensor y del actuador dentro de un solo proyecto PlatformIO,
Para construir e integrar ambas placas sin mezclar sus fuentes.

**Trazabilidad:** AD-1, AD-2, AD-5; restricciones PlatformIO del PRD; NFR-05; rúbrica de desarrollo e implementación.

**Dependencias:** Ninguna.

**Acceptance Criteria:**

**Given** el proyecto PlatformIO,
**When** se revisa su configuración,
**Then** contiene los entornos `sensor` y `actuator`, ambos con ESP32 DOIT DEVKIT V1 y framework Arduino.

**Given** los filtros `build_src_filter`,
**When** se compila cada entorno,
**Then** se incluye su propio `main.cpp` y se excluye el `main.cpp` del otro entorno.

**Given** código bajo `src/shared/`,
**When** se configura un filtro,
**Then** solo se incluye una unidad compartida si realmente la usan ambos firmwares.

**Given** el servidor TCP Python,
**When** se revisa la estructura,
**Then** está fuera de `src/` y no forma parte de la compilación PlatformIO.

**Given** cada entorno configurado,
**When** se ejecutan sus comandos de compilación y carga durante implementación,
**Then** `pio run -e sensor` y `pio run -e actuator` construyen cada firmware de forma independiente y la carga selecciona el entorno correspondiente.

### Story 1.2: Reconstruir y validar mensajes IoT-Range/1

Como integrante del grupo,
Quiero que el servidor reconstruya y valide correctamente los mensajes de línea IoT-Range/1,
Para que datos incompletos o malformados no se procesen como mediciones.

**Trazabilidad:** RF-06; IoT-Range/1 §§2–3; T-03, T-04 y T-05.

**Dependencias:** Story 1.1.

**Acceptance Criteria:**

**Given** un mensaje ASCII válido terminado en LF o CRLF,
**When** llega completo o dividido entre varias lecturas TCP,
**Then** el servidor reconstruye y procesa exactamente una línea completa.

**Given** varias líneas concatenadas en el flujo TCP,
**When** llegan en una misma lectura,
**Then** cada línea se procesa por separado y en orden.

**Given** una línea vacía, incompleta al cerrar, mayor de 256 bytes, no ASCII o con sintaxis inválida,
**When** el servidor la recibe,
**Then** la rechaza sin procesarla como mensaje.

**Given** campos ausentes, duplicados, desconocidos o números que no cumplen la gramática del protocolo,
**When** el parser valida el mensaje,
**Then** lo rechaza sin modificar mediciones ni estado del actuador.

**Given** `REGISTER`, `POST`, `ERROR` o `PING` sintácticamente válido,
**When** el parser lo acepta,
**Then** conserva verbo y campos para que el gestor aplique las reglas de rol, secuencia y sesión.

### Story 1.3: Registrar clientes TCP de forma simultánea

Como integrante del grupo,
Quiero que el servidor acepte y registre las sesiones TCP de sensor y actuador,
Para que ambos ESP32 puedan comunicarse con el servidor sin conectarse entre sí.

**Trazabilidad:** RF4, RF-05; IoT-Range/1 §3; T-01, T-02 y T-14.

**Dependencias:** Story 1.2.

**Acceptance Criteria:**

**Given** clientes con roles `sensor` y `actuator`,
**When** abren conexiones TCP independientes al host y puerto configurados,
**Then** el servidor los atiende simultáneamente y responde `ACK ref=REGISTER status=ok` a cada `REGISTER` válido.

**Given** un cliente que envía un mensaje antes de `REGISTER`, usa rol desconocido o intenta ocupar un rol ya activo,
**When** el servidor valida la sesión,
**Then** rechaza la solicitud con el error IoT-Range/1 correspondiente y no reemplaza silenciosamente otra sesión.

**Given** un registro con versión o ID inválido,
**When** el servidor lo valida,
**Then** responde con `ERROR` y cierra la sesión sin crearla.

**Given** que se inicia el servidor,
**When** se leen sus opciones de ejecución,
**Then** el puerto inicial es 5000 y el host/IP de escucha puede configurarse sin incluir credenciales WiFi en el repositorio.

### Story 1.4: Obtener lecturas válidas del HC-SR04

Como integrante del grupo,
Quiero medir distancias con el HC-SR04 y distinguir una lectura válida de una falla,
Para entregar al sistema solo datos que representen una medición real.

**Trazabilidad:** RF1, RF-07; NFR-02 y NFR-04; T-07, T-16, T-18 y T-24.

**Dependencias:** Story 1.1.

**Acceptance Criteria:**

**Given** el circuito sensor validado,
**When** se dispara una medición,
**Then** TRIG usa GPIO19 y ECHO llega a GPIO18 a través de adaptación de nivel de 5 V a 3.3 V; el firmware distingue timeout/no-echo de una distancia numérica.

**Given** un eco que representa una distancia utilizable en la ventana inicial de 5–100 cm,
**When** termina la lectura,
**Then** se devuelve una distancia finita positiva en centímetros.

**Given** timeout, eco inválido o lectura fuera del rango físico de operación declarado,
**When** termina el ciclo de medida,
**Then** se devuelve estado inválido sin conservar ni reutilizar la lectura anterior.

**Given** pruebas de lógica del driver,
**When** se inyectan timeout y pulsos representativos,
**Then** las pruebas verifican conversión y clasificación válida/inválida sin requerir una placa conectada.

### Story 1.5: Enviar mediciones y errores del sensor al servidor

Como integrante del grupo,
Quiero que el ESP32 sensor se registre y publique resultados por IoT-Range/1,
Para que el servidor reciba mediciones y pueda distinguir errores de lectura.

**Trazabilidad:** RF4, RF-05, RF-06, RF-07; IoT-Range/1 §§1–3; T-01, T-05 y T-13.

**Dependencias:** Stories 1.3 y 1.4.

**Acceptance Criteria:**

**Given** host/IP y puerto configurados,
**When** el cliente sensor se conecta,
**Then** envía `REGISTER proto=1 role=sensor id=...` antes de cualquier `POST` y valida el ACK de registro.

**Given** una lectura válida,
**When** el sensor la publica,
**Then** envía `POST seq=... distance_cm=...` con secuencia creciente, recibe `ACK ... status=accepted` y el servidor conserva la lectura válida más reciente.

**Given** timeout o lectura inválida,
**When** el sensor reporta el ciclo,
**Then** envía `ERROR code=invalid_reading ...` y no transforma la falla en distancia cero ni en distancia alta.

**Given** host servidor configurable,
**When** se cambia la configuración local,
**Then** el sensor usa el nuevo host sin recompilar valores de SSID/contraseña en el repositorio.

### Story 1.6: Recuperar la sesión del sensor y vencer datos obsoletos

Como integrante del grupo,
Quiero que el sensor se reconecte y que el servidor invalide datos vencidos,
Para que el sistema nunca use una medida antigua como si fuera actual.

**Trazabilidad:** RF-07, RF-08; protocolo §6; T-07, T-10 y T-26.

**Dependencias:** Story 1.5.

**Acceptance Criteria:**

**Given** que WiFi o TCP del sensor se interrumpe,
**When** el enlace vuelve a estar disponible,
**Then** el sensor abre sesión nueva, vuelve a enviar `REGISTER` y reinicia secuencias según IoT-Range/1.

**Given** que no llega un `POST` válido nuevo durante 1.5 s desde el último `POST` válido (o desde el registro si aún no existe una lectura válida),
**When** vence el deadman de datos,
**Then** invalida la medición vigente y registra `sensor_timeout` sin clasificarla ni reutilizarla.

**Given** que el sensor reporta `ERROR`,
**When** el servidor procesa el evento,
**Then** invalida inmediatamente la lectura previa, solicita el estado seguro y no reinicia ni extiende el deadman; únicamente un `POST` válido nuevo actualiza su referencia temporal.

**Given** que el sensor se desconecta,
**When** el servidor procesa el cierre,
**Then** invalida inmediatamente la lectura previa y solicita el estado seguro.

**Given** una sesión de sensor nueva,
**When** se procesa la reconexión,
**Then** solo un `POST` válido nuevo restablece el estado de medición.

### Story 1.7: Medir la exactitud del HC-SR04

Como integrante del grupo,
Quiero comparar las lecturas del sensor con una cinta métrica,
Para verificar el objetivo de exactitud con evidencia repetible.

**Trazabilidad:** RF1, NFR-02; T-16 y rúbrica de pruebas/resultados.

**Dependencias:** Story 1.4.

**Acceptance Criteria:**

**Given** cada distancia de referencia elegida dentro del rango inicial 5–100 cm,
**When** se realiza la medición física,
**Then** se registran cinco lecturas, cada error absoluto, el sensor/montaje y las condiciones.

**Given** las cinco lecturas de cada punto,
**When** se calcula el máximo absoluto frente a la cinta,
**Then** se compara explícitamente con el objetivo ≤3 cm y se informa aprobado o no alcanzado con los datos reales.

**Given** que un punto o distancia no puede medirse en el montaje real,
**When** se redacta el resultado,
**Then** se declara el rango efectivamente ensayado y no se extrapola a 5–100 cm.

### Story 1.8: Verificar la frecuencia de lecturas válidas

Como integrante del grupo,
Quiero medir la tasa de lecturas válidas que envía el sensor,
Para verificar el objetivo de muestreo en condiciones de eco válido.

**Trazabilidad:** RF1, RF4, NFR-04; T-18 y rúbrica de pruebas/validaciones.

**Dependencias:** Story 1.5.

**Acceptance Criteria:**

**Given** un blanco dentro de la ventana utilizable del sensor y un intervalo de prueba conocido,
**When** se contabilizan mensajes y ciclos,
**Then** se informa por separado intentos, lecturas válidas, inválidas y lecturas válidas perdidas.

**Given** la tasa de lecturas válidas calculada,
**When** se compara con el objetivo,
**Then** se declara si alcanza al menos 2 lecturas válidas por segundo, respaldado por duración y conteos.

**Given** que la tasa no alcanza el objetivo,
**When** se registra la prueba,
**Then** se reporta el resultado observado como no alcanzado y se documenta la condición, sin inventar una cifra de cumplimiento.

### Epic 2: Ver cada rango en los LEDs y demostrar una respuesta segura

El grupo puede observar el comportamiento visual correspondiente a la distancia actual mediante el recorrido completo sensor→servidor Python→actuador, demostrar límites y fallos, y preparar evidencia para informe, demo y defensa.

**FRs covered:** RF2, RF3, RF4 (cadena completa), RF-05, RF-06, RF-07, RF-08.

**Notas técnicas:** clasificación solo en Python; actuador cliente TCP con estado seguro desde reset, pinout aprobado y `SET` completo; parpadeo rojo no bloqueante; heartbeat, freshness, timeouts y reconexión IoT-Range/1; pruebas exactas automatizadas de fronteras y validación física con lecturas reales. Incluye revisión de metas NFR, diagramas y trazabilidad para los ocho apartados del informe.

**Cobertura transversal:** NFR-01–NFR-04 y pruebas físicas de integración en Epic 2; NFR-05 se verifica en ambos epics mediante estructura POO, documentación y revisión código→requisito. Las historias de Epic 2 también generan evidencia para Informe, Demo y Defensa individual (diagramas de arquitectura, dos circuitos, clases, secuencia, pruebas/resultados reales y aportes por commits). Configuración PlatformIO/hardware y entorno de servidor se preparan dentro de Epic 1 para que cada placa sea cargable independientemente.

**Dependencia y solapamiento:** Epic 1 entrega una ruta utilizable de medida→servidor. Epic 2 la extiende hasta los LEDs y completa la demo; no necesita historias futuras. La colaboración sobre el proceso servidor se divide entre recepción/validación del rol sensor en Epic 1 y clasificación/órdenes de actuador en Epic 2, evitando duplicar una misma responsabilidad.

### Story 2.1: Clasificar las distancias en Python con límites exactos

Como integrante del grupo,
Quiero que el servidor Python clasifique cada distancia válida en un único rango,
Para mantener las reglas de decisión centralizadas y verificables.

**Trazabilidad:** RF2, RF-06; reglas aprobadas del PRD; T-06.

**Dependencias:** Stories 1.2 y 1.6.

**Acceptance Criteria:**

**Given** distancia `0 < d < 20`, `20 <= d < 40` o `d >= 40`,
**When** el clasificador Python recibe el valor,
**Then** produce respectivamente el rango bajo, medio o alto sin huecos ni solapamientos.

**Given** valores exactos 19.9, 20.0, 20.1, 39.9, 40.0 y 40.1 cm,
**When** se ejecutan pruebas automatizadas,
**Then** cada uno produce la clasificación que dictan las desigualdades y las fronteras 20/40 quedan asignadas al rango superior.

**Given** cero, negativo, no finito o una lectura inválida marcada por el sensor,
**When** el clasificador recibe el dato,
**Then** no lo clasifica como distancia alta y devuelve estado inválido.

### Story 2.2: Generar y serializar comandos SET completos

Como integrante del grupo,
Quiero que Python convierta el rango y la validez de la lectura en un `SET` completo,
Para que el actuador reciba una única combinación consistente de LEDs.

**Trazabilidad:** RF2, RF3, RF-06, RF-07; IoT-Range/1 §§3–5; T-06, T-07 y T-12.

**Dependencias:** Stories 1.3, 1.6 y 2.1.

**Acceptance Criteria:**

**Given** un `POST` válido nuevo clasificado como bajo, medio o alto,
**When** el servidor crea el comando,
**Then** emite `SET` con `cmd` creciente, `cause=distance`, referencia de secuencia y campos explícitos para rojo, amarillo y verde.

**Given** una medida inválida, `ERROR`, timeout de sensor o dato vencido,
**When** el servidor genera la orden,
**Then** los tres campos LED son `off` y el comando no conserva una parte activa del estado anterior.

**Given** una medición válida en `0<d<20`, `20<=d<40` o `d>=40`,
**When** Python genera el `SET`,
**Then** asigna respectivamente `red=blink_4hz/yellow=off/green=off`, `red=off/yellow=on/green=off` o `red=off/yellow=off/green=on`.

**Given** que se reconstruye el mismo estado o cambia el rango,
**When** se serializan los `SET`,
**Then** existe un solo escritor por socket y el orden de `cmd` se mantiene sin mezclar líneas.

**Given** que el actuador no está registrado,
**When** llega un `POST`,
**Then** no se guarda un `SET` para replay; el actuador necesitará un `POST` fresco posterior a su registro.

### Story 2.3: Mantener los LEDs apagados hasta un SET válido

Como integrante del grupo,
Quiero que el actuador inicie y permanezca en estado seguro,
Para evitar una indicación falsa durante reset, arranque o datos inválidos.

**Trazabilidad:** RF3, RF-07; AD-4, AD-7; T-07, T-24 y T-25.

**Dependencias:** Story 1.1.

**Acceptance Criteria:**

**Given** el pinout aprobado,
**When** se inspecciona la configuración del actuador,
**Then** rojo usa GPIO4, amarillo GPIO21 y verde GPIO22, con resistencia serie individual de 220 o 330 Ω.

**Given** encendido, reset o inicio de firmware,
**When** los GPIO están en alta impedancia o se configura el controlador,
**Then** los tres LEDs se mantienen apagados hasta un `SET` autorizado; la prueba física verifica el circuito/polaridad que garantiza ese estado.

**Given** estado inválido, error o comando no admitido,
**When** el controlador de LEDs actualiza salidas,
**Then** aplica `SAFE_OFF` a los tres canales como una sola transición, sin estados parciales.

### Story 2.4: Aplicar y confirmar comandos IoT-Range/1 en el actuador

Como integrante del grupo,
Quiero que el actuador registre su rol, valide y aplique comandos completos,
Para representar el estado decidido por el servidor y confirmar su aplicación.

**Trazabilidad:** RF3, RF4, RF-05, RF-06, RF-07; IoT-Range/1 §§1–6; T-01, T-05, T-12 y T-23.

**Dependencias:** Stories 1.3, 2.2 y 2.3.

**Acceptance Criteria:**

**Given** el host configurado y una sesión TCP nueva,
**When** el actuador se conecta,
**Then** envía `REGISTER proto=1 role=actuator id=...` y no acepta `SET` antes del ACK de registro.

**Given** un `SET` completo, válido y con `cmd` creciente,
**When** el actuador lo aplica,
**Then** reemplaza todo el estado LED y responde `ACK ref=<cmd> status=applied`.

**Given** `SET` inválido, incompleto, con más de un LED activo o secuencia obsoleta,
**When** se recibe,
**Then** el actuador apaga los tres LEDs, envía el `ERROR` correspondiente y no aplica estado parcial.

**Given** que no llega `ACK applied` en el servidor dentro del timeout inicial de 500 ms,
**When** vence la espera,
**Then** el servidor cierra la sesión ambigua y el actuador entra en estado seguro.

### Story 2.5: Parpadear el LED rojo a 4 Hz sin bloquear TCP

Como integrante del grupo,
Quiero que el LED rojo parpadee de forma no bloqueante,
Para indicar el rango bajo y seguir atendiendo comandos de red.

**Trazabilidad:** RF3; AD-4; T-08 y T-11.

**Dependencias:** Stories 2.3 y 2.4.

**Acceptance Criteria:**

**Given** un `SET` válido para el rango bajo,
**When** el estado rojo se mantiene,
**Then** el LED rojo ejecuta cuatro ciclos completos por segundo: período 250 ms, 125 ms encendido y 125 ms apagado.

**Given** que el parpadeo está activo,
**When** llega un nuevo `SET` o un estado seguro,
**Then** el actuador lo procesa sin esperar un ciclo bloqueante y apaga los LEDs que no correspondan.

**Given** la ejecución de prueba del parpadeo y recepción TCP,
**When** se inyectan cambios de estado,
**Then** la verificación mide la cadencia y confirma que el comando se atiende dentro del límite de respuesta de 1 s.

### Story 2.6: Vigilar el heartbeat del actuador y apagar ante pérdida

Como integrante del grupo,
Quiero detectar la pérdida del servidor mediante heartbeat,
Para que los LEDs se apaguen aunque la sesión TCP quede abierta sin responder.

**Trazabilidad:** RF-07, RF-08; IoT-Range/1 §6; T-10, T-22 y T-23.

**Dependencias:** Story 2.4.

**Acceptance Criteria:**

**Given** una sesión de actuador registrada,
**When** opera normalmente,
**Then** envía `PING` con secuencia creciente cada 2 s y el servidor responde `PONG` con la secuencia correspondiente.

**Given** el actuador no recibe `PONG` correspondiente durante 6 s,
**When** vence el timeout local,
**Then** apaga los tres LEDs, cierra la sesión y reconecta de manera independiente.

**Given** el servidor no recibe `PING` durante 6 s,
**When** vence su watchdog de sesión,
**Then** cierra la sesión de actuador y no conserva comandos para replay.

**Given** reconexión del actuador,
**When** registra sesión nueva,
**Then** inicia apagado seguro y espera un `SET` de una medición fresca posterior al registro.

### Story 2.7: Integrar el recorrido nominal sensor-servidor-actuador

Como integrante del grupo,
Quiero validar el recorrido nominal de sensor a actuador,
Para demostrar que el servidor conecta las dos placas y aplica sus decisiones.

**Trazabilidad:** RF1–RF4, RF-05–RF-06; T-01, T-13 y T-14.

**Dependencias:** Stories 1.6, 2.2, 2.4 y 2.6.

**Acceptance Criteria:**

**Given** ambos ESP32 y el servidor en la WLAN,
**When** el sensor envía un `POST` válido,
**Then** el servidor valida y clasifica, envía un `SET` al actuador y el actuador confirma con `ACK applied`; la evidencia muestra el tránsito por servidor y las conexiones independientes.

**Given** una ejecución integrada,
**When** ocurren conexión, registro, medición, clasificación, comando, error o desconexión,
**Then** el servidor deja un registro comprensible de los eventos para la evidencia y el análisis de fallas.

### Story 2.8: Mantener el estado seguro ante fallas y reconexiones

Como integrante del grupo,
Quiero que errores y pérdidas de enlace apaguen los LEDs y requieran una medida fresca para reactivarlos,
Para que el actuador no mantenga ni recupere un estado basado en datos obsoletos.

**Trazabilidad:** RF-07, RF-08; IoT-Range/1 §6; T-07, T-10, T-12, T-22 y T-25.

**Dependencias:** Stories 1.6, 2.4, 2.6 y 2.7.

**Acceptance Criteria:**

**Given** `ERROR`, timeout de sensor de 1.5 s, desconexión del sensor o lectura inválida,
**When** el servidor recibe/observa la falla,
**Then** invalida la medición y envía estado completamente apagado al actuador disponible.

**Given** `SET` inválido, incompleto, de secuencia antigua o con varios LEDs activos,
**When** lo recibe el actuador,
**Then** rechaza el comando y lleva los tres LEDs a apagado sin aplicar un estado parcial.

**Given** pérdida del servidor o ausencia de `PONG` por 6 s,
**When** vence el heartbeat local,
**Then** el actuador apaga todos los LEDs y se reconecta.

**Given** sensor, actuador o servidor se reconecta,
**When** se abre una sesión nueva,
**Then** se repite `REGISTER`, no se reproduce la última medida/comando y solo una medida válida posterior al registro permite reactivar LEDs.

**Given** mensajes fragmentados/concatenados, rol incorrecto o violación de secuencia,
**When** se prueban fallas de protocolo durante la integración,
**Then** los mensajes se reconstruyen o rechazan según IoT-Range/1 y ninguna entrada inválida activa LEDs.

### Story 2.9: Verificar el tiempo de respuesta extremo a extremo

Como integrante del grupo,
Quiero medir el tiempo entre una lectura del sensor y la salida observable,
Para verificar el objetivo de respuesta declarado para el prototipo.

**Trazabilidad:** RF4, NFR-03; T-17 y rúbrica de pruebas/resultados.

**Dependencias:** Story 2.7.

**Acceptance Criteria:**

**Given** un cambio de rango y el sistema integrado en operación,
**When** se registra el instante de lectura válida y el de actuación observable,
**Then** se calcula latencia por caso y máximo observado, con método/configuración documentados.

**Given** los máximos observados,
**When** se comparan con el objetivo,
**Then** se informa explícitamente si la respuesta cumple ≤1 s; no se sustituye máximo por promedio.

**Given** una latencia superior al objetivo o datos insuficientes,
**When** se registra el resultado,
**Then** se declara la condición real y no se afirma cumplimiento sin evidencia.

### Story 2.10: Verificar estabilidad durante diez minutos

Como integrante del grupo,
Quiero operar el prototipo durante al menos diez minutos y registrar incidencias,
Para evaluar su estabilidad sin ocultar fallos.

**Trazabilidad:** NFR-01; T-15 y rúbrica de pruebas/resultados.

**Dependencias:** Story 2.7.

**Acceptance Criteria:**

**Given** la configuración integrada de demo,
**When** se inicia la prueba de estabilidad,
**Then** se registran hora inicial/final y duración continua mínima de 10 min.

**Given** el periodo de observación,
**When** ocurre reinicio, bloqueo, pérdida o recuperación,
**Then** se registra cada incidencia, su condición y efecto, separando fallas provocadas de espontáneas.

**Given** los datos de la prueba,
**When** se redacta la conclusión,
**Then** se declara el resultado observado respecto al objetivo y no se presenta el tiempo planeado como un resultado logrado.

### Story 2.11: Validar físicamente las salidas y el circuito seguro

Como integrante del grupo,
Quiero comprobar las salidas de los LEDs y el cableado con los dos ESP32 reales,
Para verificar la demo y el comportamiento eléctrico del montaje.

**Trazabilidad:** RF1–RF3, RF-07; AD-7, AD-8; CR-D1; T-08, T-09, T-20, T-21, T-24 y T-25.

**Dependencias:** Stories 2.3, 2.5, 2.7 y 2.8.

**Acceptance Criteria:**

**Given** una lectura real `0<d<20 cm`,
**When** se procesa extremo a extremo,
**Then** rojo parpadea 4 Hz y amarillo/verde permanecen apagados.

**Given** lecturas reales `20<=d<40 cm` y `d>=40 cm`,
**When** se procesan extremo a extremo,
**Then** en cada rango solo queda encendido el LED aprobado correspondiente.

**Given** lectura inválida, timeout, pérdida de enlace o reset,
**When** se observa el actuador,
**Then** los tres LEDs permanecen apagados; cada LED usa resistencia serie de 220/330 Ω y ECHO pasa por adaptación 5→3.3 V antes de GPIO18.

**Given** ensayo físico cerca de 20 y 40 cm,
**When** se documenta el resultado,
**Then** se anotan referencia, lectura real y salida resultante sin afirmar que el sensor distingue 0.1 cm; las fronteras exactas se aceptan por prueba automatizada.

**Given** el circuito montado,
**When** se inspecciona antes de alimentar,
**Then** se confirma TRIG19, ECHO18, LEDs GPIO4/21/22, polaridades, resistencias, GND común y apagado durante alta impedancia.

### Story 2.12: Documentar los diagramas técnicos exigidos

Como integrante del grupo,
Quiero disponer de diagramas claros que representen el sistema construido,
Para explicar su arquitectura, circuitos, clases y comunicación durante el informe y la defensa.

**Trazabilidad:** entregables PRD §7.1; rúbrica de informe, requerimientos/análisis/diseño; T-19, T-20 y T-24.

**Dependencias:** Stories 2.7, 2.8 y 2.11.

**Acceptance Criteria:**

**Given** la implementación y el pinout aprobado,
**When** se preparan anexos,
**Then** existen diagramas de arquitectura/despliegue, circuito sensor, circuito actuador, clases estructural y secuencia del protocolo.

**Given** el circuito sensor en el diagrama,
**When** se compara con el montaje,
**Then** muestra TRIG19, ECHO18 con adaptación 5→3.3 V, alimentación validada y GND común.

**Given** el circuito actuador en el diagrama,
**When** se compara con el montaje,
**Then** muestra GPIO4/21/22, una resistencia 220/330 Ω por LED y la condición de apagado seguro.

**Given** el diagrama de secuencia,
**When** se sigue la comunicación,
**Then** aparecen las conexiones independientes, REGISTER, POST, SET, ACK, heartbeat y al menos un flujo de error conforme a IoT-Range/1.

**Given** los diagramas finalizados,
**When** se revisan para el informe,
**Then** son legibles, consistentes entre sí y reflejan la implementación real, sin presentar esquemas conceptuales como montaje físico.

### Story 2.13: Completar el informe técnico y documentar el código

Como grupo,
Quiero entregar un informe Markdown trazable y código explicado,
Para demostrar el diseño, la implementación y las validaciones exigidas por la práctica.

**Trazabilidad:** NFR-05; PRD §§7–8; ocho secciones de informe de §4.2; criterios de rúbrica del informe.

**Dependencias:** Stories 1.7–1.8 y 2.8–2.12.

**Acceptance Criteria:**

**Given** `informe.md`,
**When** se revisa su estructura,
**Then** contiene requerimientos; análisis/diseño; desarrollo/implementación; pruebas/validaciones; resultados; conclusiones; recomendaciones; anexos.

**Given** los requerimientos oficiales y los objetivos del proyecto,
**When** se redacta el informe,
**Then** distingue requisitos del docente, decisiones del grupo, objetivos y resultados, con trazabilidad hacia pruebas y evidencia.

**Given** los resultados de pruebas efectivamente realizadas,
**When** se documentan,
**Then** incluyen datos cuantificables, análisis de errores, desviaciones, conclusiones fundamentadas y recomendaciones; pruebas no realizadas o no alcanzadas se identifican como tales.

**Given** el código C++ y Python,
**When** se explica en informe/repositorio,
**Then** se describen responsabilidades de módulos y clases, se comentan fragmentos clave y se enlazan fuentes sin copiar el código completo al informe.

**Given** la revisión de calidad,
**When** se inspecciona el código,
**Then** mantiene modularidad, orientación a objetos, documentación y convenciones adecuadas sin funcionalidades fuera de alcance.

### Story 2.14: Preparar el repositorio, evidencias y entrega académica

Como grupo,
Quiero publicar el trabajo completo en un repositorio accesible y dejar evidencia de contribución,
Para que el docente pueda revisar el proyecto y verificar la participación individual.

**Trazabilidad:** PRD §7.1; requerimientos de GitHub/LMS-UCB; rúbrica de entrega y defensa individual.

**Dependencias:** Story 2.13.

**Acceptance Criteria:**

**Given** los entregables terminados,
**When** se revisa el repositorio GitHub,
**Then** incluye informe Markdown, fuentes documentadas, diagramas, esquemas, fotografías y evidencias reales de pruebas.

**Given** cada integrante del grupo,
**When** se revisa el historial del repositorio,
**Then** se identifican commits de cada persona desde su propia cuenta GitHub y aportes explicables.

**Given** la lista del grupo,
**When** se verifica la composición,
**Then** consta de 2 o 3 integrantes como indica la práctica y cada integrante tiene una cuenta GitHub propia para sus commits.

**Given** el repositorio publicado,
**When** se verifica el acceso,
**Then** el docente puede abrir el contenido necesario antes de la fecha límite.

**Given** la entrega de la práctica,
**When** vence el martes 13 de octubre de 2026,
**Then** el enlace del repositorio se remite por LMS-UCB y el grupo conserva evidencia del envío; no se simula ni se afirma una entrega antes de realizarla.

### Story 2.15: Ensayar la demostración integrada

Como grupo,
Quiero ensayar el recorrido completo y las fallas previstas,
Para presentar una demo clara, estable y segura sin ocultar problemas de red o hardware.

**Trazabilidad:** RF1–RF4, CR-D1; rúbrica Demo §5.3; T-01, T-06–T-10, T-14, T-20–T-25.

**Dependencias:** Stories 2.8–2.11 y 2.14.

**Acceptance Criteria:**

**Given** servidor, sensor y actuador preparados,
**When** se sigue el guion de demo,
**Then** ambos ESP32 se conectan de forma independiente por WiFi/TCP y se demuestra el flujo sensor→servidor→actuador.

**Given** distancias representativas de los tres rangos,
**When** se acerca/aleja el objeto,
**Then** el LED correspondiente responde en vivo y los demás permanecen apagados; las fronteras exactas se demuestran por las pruebas automatizadas y la lectura física se informa como realmente medida.

**Given** montaje y alimentación,
**When** se ejecuta la inspección previa,
**Then** el cableado está ordenado y seguro, los resistores y adaptación de ECHO están presentes y el arranque inicia apagado.

**Given** una falla de WiFi o del prototipo durante el ensayo,
**When** se activa el plan alterno,
**Then** se muestran evidencias auténticas disponibles y se declara la falla; no se presenta una grabación como operación en vivo.

### Story 2.16: Preparar la defensa individual de cada integrante

Como integrante del grupo,
Quiero poder explicar el sistema completo y justificar mis aportes,
Para responder la defensa individual sobre redes, protocolo, código y circuitos.

**Trazabilidad:** resultados de aprendizaje §2; rúbrica Defensa individual §5.4; PRD §7.1.

**Dependencias:** Stories 2.12–2.15.

**Acceptance Criteria:**

**Given** cada integrante,
**When** realiza un ensayo individual,
**Then** puede explicar IEEE 802.11, TCP/IP, cliente-servidor, sockets y el framing/mensajes de IoT-Range/1.

**Given** el proceso de configuración y construcción,
**When** se solicita justificarlo,
**Then** puede explicar conectividad WiFi/IP, el flujo TCP y cómo se configura, compila y carga su entorno PlatformIO.

**Given** diagramas y fuentes,
**When** el docente pregunta por cualquier tramo del sistema,
**Then** el integrante puede explicar el código, la clasificación en Python, el circuito completo, los niveles eléctricos y la respuesta ante errores.

**Given** el historial GitHub de la persona,
**When** se conversa sobre su aporte,
**Then** puede identificar y justificar sus propios commits sin atribuirse cambios ajenos.

## Revisión de cobertura de historias

### Requisitos funcionales

| Requisito | Historias que lo cubren |
| --- | --- |
| RF1 | 1.4, 1.7, 2.7, 2.11, 2.15 |
| RF2 | 2.1, 2.2, 2.11, 2.15 |
| RF3 | 2.2–2.5, 2.7, 2.11, 2.15 |
| RF4 | 1.3, 1.5, 2.4, 2.7, 2.15 |
| RF-05 | 1.2, 1.3, 1.5, 2.4, 2.6 |
| RF-06 | 1.2, 1.3, 1.5, 2.1, 2.2, 2.4 |
| RF-07 | 1.4–1.6, 2.2–2.8, 2.11 |
| RF-08 | 1.6, 2.6–2.8 |

### Requisitos no funcionales y rúbrica

| Requisito/criterio | Historias que lo cubren |
| --- | --- |
| NFR-01 estabilidad ≥10 min | 2.10 |
| NFR-02 error ≤3 cm, 5 lecturas por distancia de referencia en 5–100 cm inicial | 1.7 |
| NFR-03 respuesta ≤1 s | 2.5, 2.9 |
| NFR-04 ≥2 lecturas válidas/s | 1.8 |
| NFR-05 POO, modularidad y documentación | 1.1, 1.4–1.6, 2.3–2.5, 2.13 |
| Informe: requisitos, análisis/diseño, protocolo, diagramas | 2.12, 2.13 |
| Informe: implementación POO documentada | 2.13 |
| Informe: pruebas RF/NFR y análisis de errores | 1.7, 1.8, 2.7–2.11, 2.13 |
| Informe: resultados cuantificables, conclusiones, recomendaciones y anexos | 2.8–2.13 |
| Demo: sistema completo, rangos/límites en vivo, montaje ordenado/seguro | 2.7, 2.8, 2.11, 2.15 |
| Defensa: dominio del sistema, código/circuitos y contribución por commits | 2.12, 2.14, 2.16 |
| Repositorio/GitHub/LMS-UCB y plazo oficial | 2.14 |

### Dependencias de implementación

`1.1 → 1.2 → 1.3`; `1.1 → 1.4 → 1.5 → 1.6 → 1.7`; `1.5 → 1.8`; `1.6 → 2.1 → 2.2`; `1.1 → 2.3`; `1.3 + 2.2 + 2.3 → 2.4 → 2.5/2.6`; `1.6 + 2.2 + 2.4 + 2.6 → 2.7 → 2.8 → 2.9/2.10/2.11`; `2.7 + 2.8 + 2.11 → 2.12 → 2.13 → 2.14`; `2.8–2.11 + 2.14 → 2.15 → 2.16`.

## Validación de hardware pendiente de ejecución

Pinout aprobado por el usuario. La verificación física de cableado, polaridad, resistencias, adaptación de ECHO y salida apagada durante reset queda como criterio de prueba, no como decisión abierta.
