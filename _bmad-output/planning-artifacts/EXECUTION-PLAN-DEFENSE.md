# Plan de ejecución para la defensa — Práctica 2 IoT TCP

**Fuente oficial:** [`Práctica_2.md`](../../Práctica_2.md), leída completa. **Entrega:** martes 13 de octubre de 2026. **Estado:** planificación documental aprobada; no implementar ni reportar pruebas físicas hasta cerrar la puerta documental.

Documentos de soporte: [PRD](prds/prd-Practica2-2026-10-08/prd.md), [matriz de requisitos, aceptación y rúbrica](REQUIREMENTS-ACCEPTANCE-TESTS.md), [Architecture Spine](architecture/architecture-Practica2-2026-10-08/ARCHITECTURE-SPINE.md), [IoT-Range/1](protocol/TCP-APPLICATION-PROTOCOL.md).

## 1. Base oficial y alcance

El enunciado exige un objeto sensor ultrasónico, un objeto actuador con mínimo tres LEDs o un servomotor y un servidor TCP en una PC. Ambos objetos son clientes y comienzan la conexión hacia el servidor (§3). RF1 mide; RF2 clasifica en al menos tres rangos contiguos sin solapamiento; RF3 actúa distinto por rango; RF4 intercambia medidas y comandos sobre TCP/WiFi IEEE 802.11 usando un protocolo propio (§3.1).

Se conserva la arquitectura aprobada: un proyecto PlatformIO, dos ESP32 DOIT DEVKIT V1, entornos `sensor` y `actuator`, framework Arduino, fuentes filtradas por `build_src_filter`, compartidas solo si ambas las usan, y servidor Python fuera de `src/`. El servidor decide los rangos. Umbrales/patrones LED son decisiones del grupo y deben quedar documentados; la práctica no fija los ejemplos.

## 2. Puerta documental antes de implementar

1. Revisar el PRD y distinguir mandatos oficiales de decisiones de proyecto.
2. Cerrar puntos materiales abiertos: validar físicamente el rango inicial 5–100 cm del HC-SR04, decidir asignación LED fuera de pines strapping, circuito y niveles eléctricos, roster 2–3 integrantes/cuentas, host LAN y valores de timeout/heartbeat/protocolo.
3. Revisar los diagramas propuestos de arquitectura/despliegue y clases; completar los circuitos sensor/actuador con el montaje real; verificar el diagrama de secuencia contra IoT-Range/1.
4. Congelar criterios de aceptación y métodos para todos los NFR declarados.
5. Asegurar que cada miembro entiende decisiones, circuito, clases, flujo TCP y sus propios commits.

La puerta documental se cierra antes de implementar firmware o servidor. El código no comienza en esta etapa.

## 3. Especificación de organización PlatformIO

Estructura objetivo:

```text
Practica2/
├── platformio.ini
├── src/
│   ├── sensor/main.cpp
│   ├── actuator/main.cpp
│   └── shared/                 # solo unidades usadas por ambos
├── include/                    # encabezados compartidos
├── server/                     # Python, fuera del src_dir
├── test/
├── docs/                        # circuitos/diagramas si conviene
└── informe.md                   # Markdown final
```

Configuración prevista (no aplicada):

```ini
[platformio]
default_envs = sensor, actuator

[env]
platform = espressif32
board = esp32doit-devkit-v1
framework = arduino

[env:sensor]
build_src_filter = +<sensor/> +<shared/>

[env:actuator]
build_src_filter = +<actuator/> +<shared/>
```

Compilación y carga independiente:

```sh
pio run -e sensor
pio run -e actuator
pio run -e sensor -t upload
pio run -e actuator -t upload
python3 server/main.py
```

Seleccionar el puerto serie de la placa correcta cuando ambas estén conectadas. La documentación oficial de PlatformIO permite filtrar fuentes bajo `src_dir` con `build_src_filter` y patrones include/exclude.

## 4. Protocolo y operación prevista

IoT-Range/1 queda aprobado como contrato de diseño para planificar, con parámetros iniciales sujetos a validación física/de red: líneas TCP LF/CRLF, `REGISTER`, `POST`, `SET`, `ERROR`, `ACK` y `PING/PONG` del actuador cada 2 s con timeout 6 s. El deadman de sensor vence 1.5 s después del último `POST` válido (o desde `REGISTER` si aún no existe uno); solo un nuevo `POST` válido reinicia la referencia. `ERROR` invalida inmediatamente la lectura anterior sin renovar el deadman. La práctica pide protocolo propio, pero no impone estos valores numéricos.

Contrato de actuación aprobado:

| Distancia/condición | Salida esperada |
| --- | --- |
| `0<d<20 cm` | Rojo parpadea 4 Hz; amarillo y verde apagados. |
| `20≤d<40 cm` | Amarillo fijo; rojo y verde apagados. |
| `d≥40 cm` | Verde fijo; rojo y amarillo apagados. |
| Medida inválida, error, dato obsoleto o conexión perdida | Los tres LEDs apagados. |

El servidor hace la clasificación. El actuador aplica estados sin bloquear TCP; ante pérdida de servidor apaga localmente. La nueva sesión de actuador no reutiliza un comando/medida histórica.

## 5. Plan de verificación

Las metas iniciales adoptadas (no resultados) son estabilidad ≥10 min; rango de exactitud 5–100 cm con cinco lecturas por distancia de referencia y error máximo ≤3 cm; latencia extremo a extremo ≤1 s y frecuencia ≥2 lecturas válidas/s. El resultado dependerá de la validación del sensor/montaje real.

1. **Pruebas de lógica/protocolo:** roles, mensajes inválidos, framing TCP fragmentado y coalescido, límites, secuencias, estados LED y manejo de errores. Automatizar clasificación/parser cuando sea posible.
2. **Pruebas de componente:** cada firmware y el servidor por separado; observar sensor y LEDs.
3. **Pruebas de integración:** dos clientes TCP simultáneos, WiFi común, rutas obligatorias por servidor, seis límites 19.9/20/20.1/39.9/40/40.1 cm, pérdidas y reconexiones.
4. **Métricas físicas:** para cada distancia de referencia dentro del rango inicial 5–100 cm, tomar cinco lecturas; registrar estabilidad ≥10 min, error máximo objetivo ≤3 cm, respuesta ≤1 s y tasa ≥2 lecturas válidas/s. Separar objetivo, procedimiento, datos y conclusión.
5. **Análisis de fallas:** para cada error indicar condición, respuesta observada, resultado, causa probable y corrección/recomendación si aplica.

El detalle de casos T-01–T-26 y cobertura por requisito/rúbrica está en `REQUIREMENTS-ACCEPTANCE-TESTS.md`. Todos están pendientes de ejecución; no se rellenarán resultados anticipados.

## 6. Rúbrica y preparación de defensa

### Informe (1/3)

Preparar las ocho secciones oficiales del informe: requerimientos; análisis/diseño; desarrollo/implementación; pruebas/validaciones; resultados; conclusiones; recomendaciones; anexos. El nivel 4 requiere requisitos completos y trazables, cuatro diagramas claros y protocolo completo; código modular POO documentado; plan de pruebas que cubra RF y NFR con análisis de errores; resultados cuantificables y conclusiones/recomendaciones/anexos pertinentes. El puntaje son 4 criterios × 0–4 (16 máximo).

### Demo (1/3)

Ensayar inicio del servidor, registro de clientes, flujo sensor→servidor→actuador, los tres rangos y los límites pedidos por el docente; verificar estabilidad y montaje ordenado/seguro. Tener un método alterno para mostrar evidencias si el entorno de red falla, sin ocultar la falla. Nivel global 0–4.

### Defensa individual (1/3)

Cada integrante debe poder explicar decisiones, topología, protocolo TCP/framing, clasificación, código y circuito completo, además de justificar sus decisiones. Planificar tareas y commits para que cada miembro contribuya desde su propia cuenta GitHub y pueda explicar ese aporte. Nivel global 0–4.

La defensa debe cubrir saberes de §2: IEEE 802.11/red IP, TCP/IP, cliente-servidor, sockets TCP y diseño del protocolo; la preparación y el trabajo deben evidenciar colaboración, responsabilidad y disciplina del grupo.

La nota final es el promedio de los tres componentes. El funcionamiento en vivo pertenece a Demo; no sustituye la evidencia del informe.

## 7. Publicación y fecha límite

- Confirmar que el repositorio GitHub contiene informe Markdown, código fuente documentado y anexos: diagramas, esquemas, fotografías y evidencias reales de pruebas.
- Verificar acceso del docente al repositorio antes de la fecha límite.
- Comprobar que cada integrante tiene commits desde su propia cuenta.
- Enviar enlace por LMS-UCB hasta el martes 13-10-2026; una persona del grupo puede realizar el envío.
- Mantener evidencias de acceso/envío en anexos o registro del grupo. La entrega inaccesible equivale a no entregada.

## 8. Pendientes materiales de planificación

- Definir grupo de 2 o 3 y reparto de responsabilidad/commit.
- Validar módulo HC-SR04, alimentación y rango inicial 5–100 cm; hacer cinco lecturas por distancia de referencia y comprobar error objetivo ≤3 cm.
- Cablear según pinout aprobado: TRIG GPIO19, ECHO GPIO18 con adaptación 5→3.3 V, LED rojo GPIO4, amarillo GPIO21 y verde GPIO22; una resistencia 220/330 Ω por LED y GND común. Verificar polaridad y apagado durante reset antes de energizar.
- Validar los parámetros iniciales de IoT-Range/1: PING cada 2 s, timeout 6 s, puerto TCP 5000, host configurable y deadman de 1.5 s basado solo en `POST` válido; `ERROR` invalida de inmediato sin reiniciarlo.
- Definir el host LAN estable del servidor para la red del demo; protocolo propone puerto 5000 como predeterminado configurable.
- Crear el informe final al existir implementación y resultados; mantenerlo en blanco respecto a resultados hasta medir.
