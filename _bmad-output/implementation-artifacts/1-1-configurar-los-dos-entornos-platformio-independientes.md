---
title: Configurar los dos entornos PlatformIO independientes
status: done
story_key: 1-1-configurar-los-dos-entornos-platformio-independientes
source: ../planning-artifacts/epics.md
baseline_commit: NO_VCS
context:
  - /Users/henryriveramendez/Documents/PlatformIO/Projects/Practica2/_bmad-output/planning-artifacts/prds/prd-Practica2-2026-10-08/prd.md
  - /Users/henryriveramendez/Documents/PlatformIO/Projects/Practica2/_bmad-output/planning-artifacts/architecture/architecture-Practica2-2026-10-08/ARCHITECTURE-SPINE.md
  - /Users/henryriveramendez/Documents/PlatformIO/Projects/Practica2/_bmad-output/planning-artifacts/architecture/architecture-Practica2-2026-10-08/DETAILED-ARCHITECTURE.md
  - /Users/henryriveramendez/Documents/PlatformIO/Projects/Practica2/platformio.ini
  - /Users/henryriveramendez/Documents/PlatformIO/Projects/Practica2/src/main.cpp
---

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

## Tasks & Acceptance

- [x] Actualizar `platformio.ini` para usar un solo proyecto PlatformIO, ESP32 DOIT DEVKIT V1, Arduino y los entornos `sensor` y `actuator` con filtros de fuentes separados.
- [x] Crear `src/sensor/main.cpp` y `src/actuator/main.cpp` como entradas mínimas independientes que compilen para sus respectivos entornos; no añadir lógica de sensor, red ni LEDs de historias posteriores.
- [x] Mantener el servidor Python fuera de `src/` y crear únicamente la ubicación/documentación estructural `server/`; no implementar el servidor en esta historia.
- [x] No añadir código a `src/shared/` mientras no exista una unidad que ambos firmwares realmente utilicen.
- [x] Compilar `sensor` y `actuator` por separado y comprobar mediante la salida del build que cada uno incluye su propio `main.cpp` y excluye el otro. Ver resultados exactos en **Verificación**.
- [x] Confirmar que `src/main.cpp` ya no es una unidad compilada por ninguno de los entornos.

## Verificación

- `uv run --no-cache --with platformio pio run -e sensor` — `SUCCESS`; la salida compiló `.pio/build/sensor/src/sensor/main.cpp.o` y no mostró `src/actuator/main.cpp`.
- `uv run --no-cache --with platformio pio run -e actuator` — `SUCCESS`; la salida compiló `.pio/build/actuator/src/actuator/main.cpp.o` y no mostró `src/sensor/main.cpp`.

## Review Triage Log

| Hallazgo | Veredicto y evidencia |
| --- | --- |
| Blind Hunter: documentar comandos de compilación/carga en `server/README.md`. | `false` — el README solo establece la ubicación del servidor, como pide la tarea estructural; los comandos están en la historia y se verificaron en ambos entornos. No se demostró un fallo por ausencia de esos comandos en este archivo. |
| Blind Hunter: agregar comentarios o comportamiento a las entradas vacías. | `false` — las entradas mínimas compilan y cumplen el límite explícito de no añadir lógica de historias posteriores; no se identificó un requisito de comentario o comportamiento inicial. |
| Verification Gap: no hay verificación ejecutable/build que pruebe la selección aislada. | `false` — después de la revisión se ejecutaron ambos builds y la salida mostró solo `src/sensor/main.cpp` para `sensor` y `src/actuator/main.cpp` para `actuator`; ambas compilaciones terminaron en `SUCCESS`. |
| Blind Hunter: la historia no registra los comandos exactos ni resultados suficientes para auditar la compilación. | `low; patch` — los builds terminaron correctamente, pero la ficha solo resume los comandos y la selección de archivos; agregar los comandos ejecutados y sus resultados hará verificable el criterio desde el artefacto. |
| Blind Hunter: el contexto no identifica las rutas/versiones del PRD y arquitectura. | `false` — el frontmatter `context` contiene las rutas absolutas al PRD, Architecture Spine y arquitectura detallada; sus documentos también conservan metadatos de fecha/estado. |
