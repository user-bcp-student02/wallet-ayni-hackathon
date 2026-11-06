# Proposal

## Why

HU01 — Confirmación de pagos de bajo monto no tiene todavía un recorrido extremo-a-extremo verificable: las
pantallas y la orquestación del pago están descritas en la HU original pero no como una slice entregable
por separado. **HU01-S1 — Pago exitoso de bajo monto** ([HU01-S1-pago-exitoso.md](../../../HU01-S1-pago-exitoso.md))
recorta el flujo completo a su camino feliz (sin validaciones de rango ni manejo de errores) para producir
el primer "walking skeleton": un pagador que selecciona un contacto, captura un monto, confirma y ve el
pago resuelto como Exitoso. Esta slice es prerrequisito funcional explícito de HU01-S2 (validación de
rango), HU01-S3 (resultados no exitosos) y HU01-S4 (orquestador de pago — `504`), que reutilizan las
pantallas T3/T4 y la orquestación T7 construidas aquí.

## What Changes

- Nueva pantalla de selección de contacto (T2): lista fija de contactos de mock (`contact-demo`,
  `merchant-demo`) con nombre y alias enmascarado, sin buscador.
- Nueva pantalla de datos del pago (T3): origen de solo lectura (`wallet-demo`), destino el contacto
  elegido, monto en soles capturado con teclado numérico y convertido a céntimos antes de enviarlo al
  backend.
- Nueva pantalla de confirmación previa (T4/CA1): resumen en texto simple (p. ej. "Vas a pagar S/ 15.00 a
  Juan Pérez") con botones **Confirmar** y **Corregir**.
- Nueva orquestación del pago (T7, solo camino feliz):
  - Bloqueo del botón de confirmar mientras se procesa (protección básica de doble toque).
  - Generación de un `Idempotency-Key` (1–200 caracteres, único por wallet) enviado como header
    obligatorio en `POST /payments`.
  - *Polling* de `GET /payments/{id}` tras `202 PENDING` hasta alcanzar el estado terminal `APPROVED`.
  - Mapeo de campos entre el cuerpo camelCase de `POST /payments` (`walletId`, `amountCents`) y las
    respuestas snake_case del backend (`wallet_id`, `amount`, etc.).
  - Invocación del componente de resultado de **HU01-S6** con el estado Exitoso y los datos del pago
    (`id`, fecha, destino, monto) al llegar a `APPROVED`, o con el estado "Procesando" mientras no se
    resuelve.
- Todas las pantallas se construyen sobre los componentes de UI y el catálogo de mensajes de **HU01-S0**
  (fuera de alcance de este change: ver Dependencias en la HU).
- No hay cambios de contrato existente (no hay **BREAKING**): `POST /payments` y `GET /payments/{id}` son
  endpoints nuevos consumidos por esta slice, no una modificación de un contrato previo.

## Capabilities

### New Capabilities
- `payment-request-capture`: flujo de UI por el que un pagador autenticado selecciona un contacto de la
  lista fija de mock, captura el monto del pago y confirma un resumen previo antes de enviarlo (T2, T3,
  T4/CA1).
- `payment-orchestration`: lógica que toma una solicitud de pago confirmada y la lleva a un resultado
  terminal — generación de `Idempotency-Key`, envío de `POST /payments`, *polling* de
  `GET /payments/{id}`, bloqueo de doble toque y entrega del estado resuelto (o "Procesando") a la
  pantalla de finalización (T7).

### Modified Capabilities
<!-- Ninguna: openspec/specs/ no contiene specs previas (`openspec list --specs` no encontró specs). -->

## Impact

- Código afectado: `web/` — nuevas pantallas/componentes para selección de contacto, datos del pago y
  confirmación previa, más el servicio de orquestación que integra con `POST /payments` y
  `GET /payments/{id}`. Este proposal solo produce los artefactos de planeación (specs, design, tasks); no
  modifica código todavía.
- APIs: consume `POST /payments` (requiere header `Idempotency-Key`) y `GET /payments/{id}` para polling.
  Estos endpoints no existen hoy en `api/app/main.py` (que solo expone `/auth` y `/tasks`); su
  implementación (real o mock) es una dependencia externa a esta slice y debe resolverse antes de poder
  implementar HU01-S1 — queda registrada como pregunta abierta para `design.md`.
- Dependencias funcionales: requiere los componentes y catálogo de mensajes de **HU01-S0**, el punto de
  entrada de **HU01-S5** (Home → "Nuevo pago") y el componente de presentación de resultado de
  **HU01-S6**; ninguna de esas tres HU se modifica en este change (se consumen tal como están
  especificadas).
- Verificación: pruebas de caracterización del escenario mock `APPROVED` (camino feliz completo) y del
  bloqueo de doble toque, por definir en `tasks.md`; meta de cobertura del código modificado ≥ 85%.
