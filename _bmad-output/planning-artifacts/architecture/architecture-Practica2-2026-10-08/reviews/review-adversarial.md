# Revisión adversarial de arquitectura

## Hallazgos y resolución

- **P1 — Frescura de medición.** Se explicitó en AD-4 que los datos vencen 1.5 s después del último `POST` válido y que esto es independiente del heartbeat. Se invalidan y el sistema apaga.
- **P1 — Arranque seguro.** Se añadió al diseño detallado el criterio de LEDs apagados desde reset hasta `SET` autorizado, incluso con GPIO en alta impedancia; verificación eléctrica pendiente (T-25).
- **P1 — Reasignación de pines.** AD-7 declara GPIO2/GPIO15 no recomendados para LEDs y propone GPIO21/GPIO22. Requiere confirmación y cotejo de la placa concreta; por ello no se cierra el pinout físico.
- **P2 — Semántica heartbeat.** AD-6 especifica servidor esperando `PING` y actuador esperando `PONG` correspondiente, ambos con 6 s; sensor no usa ese heartbeat.
- **P2 — Temporizadores.** El diseño detallado reúne deadman 1.5 s, heartbeat 2/6 s, ACK 500 ms y objetivo respuesta 1 s.
- **P2 — Concurrencia.** AD-9 fija un coordinador, sesiones simultáneas y cola serial de `SET`; se propone bucle `selectors` de la biblioteca estándar.

No se detectaron contradicciones con PlatformIO de dos entornos ni clasificación central en Python.

## Resolución posterior

El usuario aprobó el pinout recomendado: LED amarillo GPIO21 y verde GPIO22. La decisión AD-7 queda adoptada en la arquitectura; queda pendiente únicamente su verificación física en la placa y circuito reales.
