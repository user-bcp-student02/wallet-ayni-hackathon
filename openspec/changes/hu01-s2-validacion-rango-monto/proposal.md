# Proposal

## Why

**HU01-S1 — Pago exitoso de bajo monto** sólo cubre el camino feliz de T3/T4: cualquier monto capturado se
envía directamente a `POST /payments` sin validar que esté dentro del rango de "bajo monto". Sin esta
validación, un pagador puede llegar a la confirmación previa (T4) y enviar el pago con un monto que el
servidor rechazará de todas formas, generando fricción evitable y el riesgo de dejar al pagador en un
estado "Procesando" falso cuando la discrepancia se detecta recién en el servidor. **HU01-S2 — Validación
de rango de bajo monto** ([HU01-S2-validacion-rango-monto.md](../../../HU01-S2-validacion-rango-monto.md))
cierra ese vacío cubriendo íntegramente **CA2** y **T8** de la HU01 original.

## What Changes

- Validación en **T3 (Datos del pago)**: antes de habilitar "Continuar" hacia T4, el sistema bloquea el
  envío si el monto ingresado es menor a S/ 10 o mayor a S/ 1000, mostrando el mensaje de bloqueo del
  catálogo de **HU01-S0**.
- Piso técnico de captura en el campo numérico de monto: rechaza a nivel de entrada cualquier valor menor a
  S/ 0.01, de forma independiente a la regla de negocio de S/ 10–S/ 1000.
- Validación de defensa en profundidad en el servidor al orquestar `POST /payments` (construido en
  HU01-S1): si un monto fuera de rango llega a esta llamada pese a la validación de la app, el servidor lo
  rechaza y el pagador recibe el mismo mensaje de bloqueo claro, sin llegar a un estado "Procesando" falso.
- No hay cambios de contrato existente (no hay **BREAKING**): no se modifican los campos ni el formato de
  `POST /payments`; se añade una regla de validación sobre un monto que ese endpoint ya recibía.

## Capabilities

### New Capabilities
- `payment-amount-range-validation`: regla de negocio que valida que el monto de un pago de bajo monto
  esté entre S/ 10 y S/ 1000 (inclusive), aplicada tanto en la app (antes de avanzar de T3 a T4) como en el
  servidor al orquestar el pago (defensa en profundidad), más el piso técnico de captura de S/ 0.01 en el
  campo numérico.

### Modified Capabilities
<!-- openspec/specs/ aún no contiene specs sincronizadas: `openspec list --specs` no encontró ninguna.
     Las capabilities `payment-request-capture` y `payment-orchestration` de HU01-S1 existen solo como
     delta pendiente de sync en openspec/changes/hu01-s1-pago-exitoso/specs/, no como spec canónica, por lo
     que no hay un path existente bajo openspec/specs/ que modificar todavía. -->

## Impact

- Código afectado: `web/` — lógica de validación en la pantalla T3 (bloqueo de "Continuar" y piso técnico
  del campo numérico), reutilizando el catálogo de mensajes de HU01-S0. `api/` (o el mock que materialice
  `POST /payments` para HU01-S1) — regla de validación de rango como defensa en profundidad antes de
  orquestar el pago. Este proposal solo produce los artefactos de planeación (specs, design, tasks); no
  modifica código todavía.
- APIs: no agrega ni modifica el contrato de `POST /payments`; añade una condición de rechazo sobre el
  monto que ese endpoint ya recibe (definido como parte de la orquestación de HU01-S1).
- Dependencias funcionales: requiere las pantallas T3/T4 y la orquestación `POST /payments` de **HU01-S1**,
  y el catálogo de mensajes de **HU01-S0** para el texto de bloqueo; no es prerrequisito de HU01-S3 ni
  HU01-S4 (son independientes entre sí).
- Verificación: pruebas de caracterización en los bordes del rango (S/ 10, S/ 9.99, S/ 0.01, S/ 1000,
  S/ 1000.01) y del caso de discrepancia cliente/servidor, por definir en `tasks.md`; meta de cobertura del
  código modificado ≥ 85%.
