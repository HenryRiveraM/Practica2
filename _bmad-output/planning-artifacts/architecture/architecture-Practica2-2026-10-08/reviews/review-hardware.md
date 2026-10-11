# Revisión de compatibilidad eléctrica

- GPIO2 y GPIO15 son strapping pins; GPIO15 también tiene función JTAG. Evitar ambos para LED es una recomendación adecuada.
- GPIO4, GPIO18, GPIO19, GPIO21 y GPIO22 aparecen como GPIO utilizables en el ESP32 clásico. GPIO21/22 son propuesta pendiente de aceptación y de cotejo con serigrafía/revisión DOIT V1.
- Se afinó el texto de entrada: con VDD=3.3 V, `VIH` máximo especificado es VDD+0.3 V (3.6 V). ECHO potencial de 5 V requiere adaptación, sin asignar valores antes de identificar módulo.
- Se aclaró el sentido de conexión ECHO→adaptador→GPIO18. El umbral TRIG del HC-SR04 real permanece a verificar.
- El criterio de apagado durante reset se sujeta a inspección de la topología/polaridad y configuración eléctrica de GPIO en alta impedancia.

Fuentes oficiales: Espressif GPIO reference, hardware schematic checklist y ESP32 datasheet (citadas en `DETAILED-ARCHITECTURE.md`).

## Resolución posterior

El usuario aprobó GPIO21 para LED amarillo y GPIO22 para LED verde, manteniendo rojo GPIO4, TRIG19 y ECHO18 con adaptación 5→3.3 V. La revisión física del módulo/placa, resistores, polaridad y apagado en reset sigue como criterio pendiente de prueba.
