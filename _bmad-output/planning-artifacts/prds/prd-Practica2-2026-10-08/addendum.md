# Addendum técnico del PRD

Este archivo conserva decisiones de implementación y contratos de integración, que el PRD mantiene fuera de los requisitos de capacidad. No habilita todavía la implementación.

## Artefactos fuente que gobiernan el diseño

- `../../architecture/architecture-Practica2-2026-10-08/ARCHITECTURE-SPINE.md`: AD-1 a AD-5 aprobadas y estructura PlatformIO de un proyecto con dos entornos.
- `../../protocol/TCP-APPLICATION-PROTOCOL.md`: borrador IoT-Range/1, framing, roles, mensajes, heartbeat, fallos y salida segura.
- `../../REQUIREMENTS-ACCEPTANCE-TESTS.md`: matriz RF/NFR, pruebas y correspondencia con rúbrica.
- `../../EXECUTION-PLAN-DEFENSE.md`: secuencia de documentación, integración y preparación de defensa.

## Estructura de desarrollo aprobada

```text
Practica2/
├── platformio.ini
├── src/sensor/main.cpp
├── src/actuator/main.cpp
├── src/shared/                 # solo si realmente se comparte
├── include/
├── server/                     # Python, fuera del src_dir de PlatformIO
├── test/
└── informe.md                  # informe técnico final, después de las pruebas
```

`[env:sensor]` y `[env:actuator]` usarán el mismo ESP32 DOIT DEVKIT V1 + Arduino, cada uno con `build_src_filter` propio. Los comandos planeados son `pio run -e sensor`, `pio run -e actuator`, y sus cargas individuales con `-t upload`. Nada se ha aplicado al código ni probado.

## Secuencia lógica de integración

1. Sensor y actuador se conectan a la misma WLAN y abren cada uno TCP hacia el host LAN del servidor Python.
2. Cada placa se registra como un rol mediante `REGISTER`.
3. El sensor envía medidas válidas con `POST` e informa fallos de lectura mediante `ERROR`.
4. Python valida, clasifica centralmente y genera una asignación completa `SET` para los tres LEDs.
5. El actuador aplica el estado sin bloquear la lectura TCP y confirma la aplicación mediante `ACK`.
6. Timeout, medida inválida, comando inválido, dato obsoleto o pérdida de enlace llevan a los tres LEDs apagados.

## Protocolo: cuestiones a cerrar antes de implementación

- Heartbeat inicial: `PING` cada 2 s; timeout de heartbeat: 6 s. El sensor tiene un deadman separado de 1.5 s: solo un `POST` válido nuevo reinicia la referencia; `ERROR` invalida inmediatamente la lectura previa pero no reinicia ni extiende el deadman. Si aún no hubo `POST` válido, el deadman corre desde `REGISTER`. Validar tiempos en la red real junto con la meta de respuesta ≤1 s.
- Reconfirmar en el hardware el rango válido de distancia y el circuito del HC-SR04.
- Pinout aprobado: TRIG GPIO19, ECHO GPIO18 con adaptación 5→3.3 V, LED rojo GPIO4, amarillo GPIO21 y verde GPIO22. No usar GPIO2/GPIO15 para LEDs porque son strapping pins del ESP32 clásico.
- Usar resistencias limitadoras de 220/330 Ω para cada LED y adaptar el nivel del ECHO antes de GPIO18. Comprobar la variante de módulo HC-SR04, tierra común, tensión de ECHO y operación de TRIG antes de energizar; no fijar valores del divisor hasta la verificación del sensor.
- Metas aprobadas para medir: rango inicial 5–100 cm, cinco repeticiones por distancia de referencia, error absoluto máximo 3 cm, al menos 2 lecturas válidas/s, respuesta ≤1 s y estabilidad ≥10 min. Son objetivos pendientes, no resultados.
- Automatizar fronteras exactas del clasificador. La prueba física usa lecturas reales y no presupone discriminación de décimas de centímetro.
- Establecer la dirección LAN del servidor y documentar cómo configurarla sin guardar credenciales WiFi en el repositorio.
