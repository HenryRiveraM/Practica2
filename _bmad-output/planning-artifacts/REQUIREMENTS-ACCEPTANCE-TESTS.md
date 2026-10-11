# Requerimientos, aceptación, rúbrica y pruebas

**Fuente oficial leída completa:** [`Práctica_2.md`](../../Práctica_2.md), §§2–6. Este documento distingue obligaciones oficiales, metas cuantitativas adoptadas por el grupo y decisiones internas. El estado de cada prueba es **Pendiente**; no se han ejecutado ni se declaran resultados.

## 0. Resultados de aprendizaje oficiales (§2)

- **Conceptual:** redes de sensores IP sobre IEEE 802.11, TCP/IP, cliente-servidor y TCP.
- **Procedimental:** configurar conectividad IP/WiFi, implementar sockets TCP y diseñar/documentar protocolo simple de aplicación.
- **Actitudinal:** colaborar, asumir responsabilidades y trabajar de forma ordenada/sistemática. La rúbrica de defensa verifica contribución individual mediante commits propios.

## 1. Requerimientos funcionales

### 1.1 Requerimientos oficiales (§3, §3.1)

| ID local / ID oficial | Requisito | Criterio de aceptación | Verificación planeada |
| --- | --- | --- | --- |
| RF-01 / RF1 | Medir la distancia entre el sensor ultrasónico y un objeto. | El ESP32 sensor realiza muestras dentro del rango de trabajo declarado y distingue valor medible de timeout/lectura inválida. | Prueba física con distancias conocidas y sin eco. |
| RF-02 / RF2 | Interpretar la distancia en al menos tres rangos contiguos y sin solapamiento. | Cada distancia válida dentro del rango cae en exactamente un intervalo; se usan las fronteras aprobadas 20 y 40 cm. | Pruebas de límites 19.9, 20, 20.1, 39.9, 40 y 40.1 cm; pruebas unitarias de clasificación. |
| RF-03 / RF3 | Activar el actuador de forma distinta según el rango detectado. | Bajo: rojo parpadea a 4 Hz; medio: amarillo fijo; alto: verde fijo; los otros LEDs apagados; inválido/error: los tres apagados. | Prueba física y medición temporal de parpadeo. Los umbrales y patrón son decisiones del proyecto, no valores de ejemplo oficial. |
| RF-04 / RF4 | Enviar lecturas sensor→servidor y comandos servidor→actuador mediante sockets TCP sobre WiFi IEEE 802.11, con protocolo propio. | Ambos ESP32 inician conexiones independientes al servidor; logs/evidencia muestran recorrido por servidor, sin conexión directa entre placas; mensajes siguen IoT-Range/1. | Integración con servidor en PC y evidencia de sesiones/mensajes. |

### 1.2 Requisitos de proyecto que completan el comportamiento

Estos puntos implementan decisiones explícitas del usuario o completan el protocolo acordado; no se presentan como RF textuales del docente.

| ID | Requisito del proyecto | Criterio de aceptación | Cobertura |
| --- | --- | --- | --- |
| PR-01 | El servidor Python acepta el objeto sensor y el actuador como clientes TCP simultáneos e identifica sus roles. | Un `REGISTER` válido obtiene ACK; rol incorrecto, sesión duplicada o mensaje previo al registro se rechaza. | T-01, T-02 |
| PR-02 | El protocolo separa y reconstruye mensajes sobre el flujo TCP. | Fragmentos se ensamblan hasta LF; varias líneas se procesan por separado; línea incompleta nunca se procesa. | T-03, T-04 |
| PR-03 | El servidor valida valores, roles y comandos antes de utilizarlos. | Valores inválidos, secuencia vieja, campo ausente/duplicado/desconocido y comando LED no permitido se rechazan. | T-05, T-06 |
| PR-04 | Las fallas llevan a estado seguro. | Timeout/no-echo/medida inválida, comando inválido, dato obsoleto y desconexiones causan apagado de los tres LEDs; ninguna lectura inválida se interpreta como distancia grande. | T-07–T-10 |
| PR-05 | Los clientes se recuperan mediante reconexión independiente. | Tras reconexión se registran de nuevo; actuador permanece apagado hasta recibir una medida fresca válida; no se reutiliza estado obsoleto. | T-09, T-10 |
| PR-06 | El parpadeo rojo no bloquea TCP. | Durante el parpadeo, el actuador procesa un `SET` nuevo dentro del presupuesto de respuesta definido. | T-11 |

## 2. Requerimientos no funcionales

`Práctica_2.md` §3.2 exige que el grupo declare y verifique valores medibles; los valores entre paréntesis son referencias, no límites exactos impuestos. El proyecto adopta inicialmente esas referencias como objetivos propios. Cumplimiento **Pendiente de medición**.

| ID | Atributo | Objetivo adoptado | Criterio / evidencia de aceptación | Fuente |
| --- | --- | --- | --- | --- |
| NFR-01 | Estabilidad y confiabilidad | ≥10 min continuos, sin reinicios ni bloqueos no esperados. | Registrar duración, reinicios, bloqueos y desconexiones. Pasar solo si el intervalo y criterio se sostienen en la ejecución física. | Meta del proyecto a partir de referencia §3.2; verificar en T-15. |
| NFR-02 | Exactitud de medición | Error absoluto máximo ≤3 cm contra cinta métrica en rango inicial 5–100 cm. Cinco lecturas por cada distancia de referencia. | Registrar cada distancia de referencia y las cinco lecturas, error absoluto por lectura, error máximo, sensor/montaje y condiciones. Si el montaje no puede cubrir la ventana completa, documentar la limitación y el rango ensayado antes de afirmar cumplimiento. | Meta adoptada por el grupo; T-16. |
| NFR-03 | Tiempo de respuesta | ≤1 s desde la medición/cambio de rango hasta la salida del actuador. | Medición extremo a extremo mediante marcas de tiempo, condiciones y máximo observado. | Meta del proyecto a partir de referencia §3.2; T-17. |
| NFR-04 | Frecuencia de muestreo | ≥2 lecturas válidas por segundo en condiciones que permitan eco válido. | Contabilizar lecturas válidas reportadas durante intervalo conocido; informar por separado intentos de medición, válidas, inválidas y pérdidas. | Meta del proyecto a partir de referencia §3.2; T-18. |
| NFR-05 | Calidad de código | Legible, modular, orientado a objetos, documentado y conforme a convenciones de C++/Python. | Revisión de estructura/clases, responsabilidades, documentación, convenciones y correspondencia de implementación con requisitos. | Mandato §3.2; revisión de implementación futura. |

El montaje ordenado/seguro se gestiona como criterio de demo **CR-D1** (§5.3), no como atributo NFR de §3.2.

## 3. Pruebas de aceptación

| Test ID | Requisitos | Procedimiento / condición de aprobación | Tipo | Estado |
| --- | --- | --- | --- | --- |
| T-01 | RF-04, PR-01 | Conectar dos clientes a la vez; ambos inician TCP al servidor y se registran con rol correcto; no hay canal placa-a-placa. | Integración | Pendiente |
| T-02 | PR-01, PR-03 | Probar roles desconocidos, cliente sin `REGISTER`, segundo cliente de mismo rol y re-registro tras reconexión. | Automatizable/socket | Pendiente |
| T-03 | RF-04, PR-02 | Dividir una línea en varios envíos TCP y concatenar dos líneas en un envío; reconstruir ambos mensajes correctamente. | Automatizable/socket | Pendiente |
| T-04 | PR-02 | Cerrar socket con línea incompleta; confirmar que no se procesa como mensaje. Probar línea vacía y límite de tamaño. | Automatizable/socket | Pendiente |
| T-05 | PR-03 | Probar campos ausentes, duplicados, desconocidos, valores no finitos, separadores decimales inválidos y bytes fuera de ASCII. | Automatizable/parser | Pendiente |
| T-06 | RF-02, RF-03 | Inyectar al clasificador exactamente 19.9, 20.0, 20.1, 39.9, 40.0 y 40.1 cm; cada valor produce el rango/salida definidos por las desigualdades. | Unit/servidor | Pendiente |
| T-07 | RF-01, PR-04 | Forzar timeout/no-echo, distancia 0/negativa/no finita y `ERROR invalid_reading`; comprobar salida totalmente apagada. | Unit + físico | Pendiente |
| T-08 | RF-03 | Distancia `0<d<20`: medir 4 ciclos/s (período 250 ms; 125 ms ON/125 ms OFF); los otros LEDs apagados. | Física | Pendiente |
| T-09 | RF-03, PR-04 | Probar físicamente valores interiores de los tres rangos; comparar distancia de cinta, lectura POST recibida y salida que corresponde a esa lectura; LEDs restantes apagados. | Física | Pendiente |
| T-10 | RF-04, PR-04, PR-05 | Desconectar por separado sensor, actuador y servidor; comprobar apagado, detección de pérdida, reconexión y registro nuevo; no reactivar con medida antigua. | Integración/física | Pendiente |
| T-11 | PR-06 | Durante blink enviar un comando que cambie a verde o estado seguro; confirmar recepción/aplicación dentro de ≤1 s. | Integración/física | Pendiente |
| T-12 | PR-03, PR-04 | Enviar `SET` con dos LEDs activos, nombre/valor desconocido y secuencia obsoleta; rechazar y conservar/aplicar estado seguro. | Automatizable + física | Pendiente |
| T-13 | RF-04 | Confirmar que la consola registra conexión, registro, medida, clasificación, comando, error y desconexión de forma comprensible. | Integración | Pendiente |
| T-14 | RF-04 | Demostrar que el host PC está en WLAN común y ambos ESP32 son clientes TCP independientes; guardar captura/foto/log pertinente. | Demo | Pendiente |
| T-15 | NFR-01 | Operar ≥10 min; registrar hora inicial/final y cualquier reinicio, bloqueo o pérdida. | Física | Pendiente |
| T-16 | RF-01, NFR-02 | En distancias de referencia distribuidas dentro del rango inicial 5–100 cm, tomar cinco lecturas por punto; registrar referencia, cada lectura, error absoluto y máximo. Si el rango físico disponible se reduce, declarar el intervalo realmente ensayado y no extrapolar. | Física | Pendiente |
| T-17 | RF-04, NFR-03 | Marcar tiempo de lectura y actuación observable por caso; reportar latencia máxima y cumplir ≤1 s. | Física | Pendiente |
| T-18 | RF-01, NFR-04 | Medir lecturas válidas por segundo en condiciones con eco válido; reportar por separado intentos, lecturas válidas, inválidas y pérdidas. Aceptar si lecturas válidas ≥2/s. | Física | Pendiente |
| T-19 | NFR-05 | Revisar arquitectura de clases, modularidad, documentación, convenciones y trazabilidad código→RF. | Inspección | Pendiente |
| T-20 | CR-D1, rúbrica Demo §5.3 | Comprobar alimentación, conexiones, resistencias, orden y montaje antes de la demo. | Inspección física | Pendiente |
| T-21 | RF-02, RF-03, NFR-02 | Si se ensaya físicamente cerca de 20 y 40 cm, registrar distancia de referencia, lectura real recibida y salida esperada según lectura. No inferir capacidad de separar físicamente 19.9 de 20.1 cm bajo tolerancia de ±3 cm; los seis valores exactos se cubren con T-06. | Física, exploratoria | Pendiente |
| T-22 | RF-03, PR-04, PR-05 | Desconectar o detener el servidor cuando un LED esté activo; verificar que, al cumplirse el timeout inicial de 6 s sin `PONG`, el actuador apaga los tres LEDs, reconecta y permanece en estado seguro hasta un `SET` generado por lectura fresca. | Integración/física | Pendiente |
| T-23 | RF-04, PR-01, PR-02 | Confirmar `REGISTER` al inicio de cada sesión, puerto TCP 5000 configurable, PING cada 2 s y timeout heartbeat a 6 s; registrar marcas de tiempo y comprobar efecto al perder PONG. | Automatizable + integración | Pendiente |
| T-24 | NFR-05, CR-D1 | Inspeccionar cableado: HC-SR04 TRIG→GPIO19; ECHO→adaptación 5→3.3 V→GPIO18; LEDs rojo/amarillo/verde en GPIO4/21/22 con resistencia serie 220/330 Ω cada uno; GND común; comprobar serigrafía y arranque en DOIT V1 real. | Inspección eléctrica | Pendiente |
| T-25 | PR-04, RF-03 | Energizar/resetear el actuador con el montaje final y observar durante setup, espera de WiFi/REGISTER y espera de dato fresco; aprobar solo si todos los LEDs permanecen apagados hasta el primer `SET` autorizado. | Inspección eléctrica + integración | Pendiente |
| T-26 | PR-04, PR-05 | En prueba automatizada del servidor: tras un `POST` válido, enviar `ERROR` y verificar invalidación/apagado inmediato; comprobar que la marca del deadman no se reinicia. Repetir `ERROR` antes y después del plazo original y verificar que ninguno extiende la frescura; solo un `POST` válido posterior inicia una nueva ventana de 1.5 s. | Automatizable/protocolo | Pendiente |

## 4. Entregables, informe y evidencia

| Obligación | Criterio de aceptación | Destino/evidencia |
| --- | --- | --- |
| Prototipo (§4.1) | Sensor, servidor y actuador operan de extremo a extremo. | Demo y anexos de pruebas. |
| Informe Markdown (§4.2) | Están presentes las ocho secciones: RF/RNF; análisis y diseño; desarrollo/implementación; pruebas; resultados; conclusiones; recomendaciones; anexos. | `informe.md` (nombre final a decidir). |
| Diagramas (§4.2) | Arquitectura/despliegue; circuito de cada objeto; clases estructural; secuencia con mensajes. | Informe o anexos enlazados. Diagramas deben corresponder al montaje real. |
| Desarrollo (§4.2) | Módulos y clases explicados; fragmentos clave comentados; fuentes enlazadas sin pegar el código completo. | Informe + código en repo. |
| Pruebas y resultados (§4.2) | Plan y resultados distinguidos; datos cuantificables, error/análisis y evidencia reales. | Informe + logs/tablas/fotos pertinentes. |
| Repositorio (§6) | Contiene informe, código documentado y anexos; docente tiene acceso antes del corte. | GitHub. |
| Aporte individual (§6) | Cada miembro tiene commits con su cuenta propia. | Historial GitHub. |
| Entrega (§6) | Link del repositorio remitido por LMS-UCB a más tardar el martes 13-10-2026; basta una entrega por grupo. | Confirmación del envío por el grupo. |

## 5. Trazabilidad con la rúbrica

| Rúbrica oficial | Nivel 4 requiere (§) | Artefactos/secciones que lo evidencian | Pruebas/evidencia |
| --- | --- | --- | --- |
| Informe: Requerimientos, análisis y diseño (§5.2.2) | RF/RNF completos, medibles y trazables; diagramas arquitectura, ambos circuitos, clases, secuencia; protocolo completo. | PRD §§4–6; arquitectura; protocolo; informe §§1–2. | Matriz RF/NFR y T-01–T-26; revisión de diagramas respecto al montaje. |
| Informe: Desarrollo e implementación (§5.2.2) | Código modular POO, documentado, buenas prácticas y coherente con todos los requisitos. | Informe §3, estructura y fuentes en GitHub. | T-19 + revisión de enlaces/código antes de entrega. |
| Informe: Pruebas y validaciones (§5.2.2) | Plan completo contra RF y valores NFR declarados, documentación detallada y análisis de errores. | Informe §4; esta matriz; anexos de medición/logs. | T-01–T-26 y tabla real de resultados/incidencias. |
| Informe: Resultados/conclusiones/recomendaciones (§5.2.2) | Resultados cuantificables, análisis crítico, conclusiones fundamentadas, recomendaciones y anexos pertinentes. | Informe §§5–8, anexos con datos/evidencias. | T-15–T-18; revisión de coherencia datos→conclusión. |
| Demo (§5.3) | Operación completa/estable vía servidor TCP/WiFi; rangos y límites pedidos en vivo; montaje ordenado/seguro. | Prototipo y hoja de ensayo de defensa. | T-06–T-10, T-14, T-20–T-26 y cualquier estímulo adicional del docente. |
| Defensa individual (§5.4) | Explicación precisa de cualquier parte de código/circuito, justificación técnica y aporte claro por commits propios. | Diagrama de clases/circuitos, PRD/arquitectura/protocolo y commits individuales. | Ensayo oral individual; comprobar autoría y trazabilidad de commits. |

## 6. Niveles completos de la rúbrica oficial

La evaluación global pondera Informe, Demo y Defensa Individual con 1/3 cada uno (§5.1). Informe = 4 criterios × 0–4; porcentaje del informe = puntos/16×100. Demo y Defensa = nivel global 0–4, convertido a porcentaje con nivel/4×100. La nota final es el promedio de los tres componentes. El nivel 0 aplica cuando no se entrega, presenta o puede evidenciar el trabajo.

### 6.1 Informe técnico (§5.2)

| Criterio | Nivel 4 — Excelente | Nivel 3 — Satisfactorio | Nivel 2 — Con recomendaciones | Nivel 1 — Necesita mejorar | Nivel 0 |
| --- | --- | --- | --- | --- | --- |
| Requerimientos, análisis y diseño | RF/RNF completos, medibles y trazables; arquitectura, dos circuitos, clases y secuencia; protocolo completo; diagramas explican sistema/ejecución. | Requerimientos definidos y diagramas claros para el funcionamiento general. | Requisitos imprecisos o diagramas incompletos/poco claros. | Falta un diagrama esencial o no se definen requisitos. | Sección no presentada. |
| Desarrollo e implementación (código/documentación) | Código modular POO, documentado, buenas prácticas; implementación cubre requisitos declarados. | Código estructurado/coherente, documentación básica. | Organización deficiente o sin convenciones claras. | Código incompleto, sin documentación o no corresponde. | No se entrega fuente. |
| Pruebas y validaciones | Plan completo contra RF y valores NFR, documentación detallada y análisis de errores cuando aplica. | Plan adecuado con documentación básica. | Plan incompleto/validaciones parciales. | Pruebas insuficientes. | No documenta pruebas. |
| Resultados, conclusiones y recomendaciones | Resultados cuantificables, análisis crítico, conclusiones fundamentadas, recomendaciones técnicas y anexos pertinentes. | Resultados medibles y conclusiones coherentes. | Resultados limitados, conclusiones básicas y sin recomendaciones. | Resultados poco claros o no medibles. | Sección no presentada. |

### 6.2 Demo (§5.3)

| Nivel | Descriptor |
| --- | --- |
| 4 | Prototipo completo y estable; comunicación vía servidor TCP/WiFi; respuesta en rangos y límites solicitados en vivo; montaje ordenado y seguro. |
| 3 | Cumple requerimientos funcionales, con fallas menores que no impiden la demo. |
| 2 | Funciona parcialmente; algún rango o actuador falla. |
| 1 | No funcional o con falla crítica durante la demo. |
| 0 | No se presenta. |

### 6.3 Defensa individual (§5.4)

| Nivel | Descriptor |
| --- | --- |
| 4 | Responde con precisión/profundidad, explica cualquier parte del código y circuito, justifica decisiones y evidencia aporte claro por sus commits. |
| 3 | Explica correctamente el conjunto con dudas puntuales. |
| 2 | Conoce solo su parte y explica superficialmente. |
| 1 | No logra explicar funcionamiento ni justificar diseño. |
| 0 | No se presenta. |

El funcionamiento en vivo se evalúa en Demo, no en la puntuación del Informe (§5.2.1); deben prepararse evidencias separadas para cada componente de evaluación.

## 7. Reglas de reporte de resultados

- Estado actual de todas las pruebas: **Pendiente**.
- No rellenar valores, fotografías, capturas, latencias, exactitud, estabilidad ni funcionamiento observados hasta ejecutar pruebas reales.
- Para cada prueba registrar fecha, configuración, distancia de referencia/observada cuando aplique, método, repeticiones, resultado, error, evidencia y análisis de fallas.
- Una meta NFR no se marca como cumplida por tener criterio de aceptación; debe haber datos que demuestren el resultado.
