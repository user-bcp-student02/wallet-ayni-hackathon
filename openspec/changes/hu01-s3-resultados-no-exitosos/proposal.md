# Proposal

## Why

HU01-S1 — Pago exitoso de bajo monto construye la orquestación del pago (T7) y el *polling* de
`GET /payments/{id}`, pero solo cubre el camino feliz hasta `APPROVED`: hoy no existe un contrato
especificado para qué le pasa al pagador cuando `POST /payments` responde con un error síncrono o cuando el
*polling* no resuelve un estado terminal a tiempo. Sin esa especificación, un pagador cuyo pago falla o
tarda demasiado queda potencialmente bloqueado en la pantalla de "Procesando", sin saber qué pasó con su
dinero ni cómo continuar. **HU01-S3 — Resultados no exitosos y mapeo de errores síncronos**
([HU01-S3-resultados-no-exitosos.md](../../../HU01-S3-resultados-no-exitosos.md)) cierra ese vacío
cubriendo **CA3** (ramas Fallido/Expirado), **CA4** (acción tras un resultado no exitoso) y la parte de
**T7** referida al mapeo de errores síncronos y al timeout propio de la app. Además, es prerrequisito
explícito de **HU01-S4** (caso especial `504`), que ya existe como change de OpenSpec y depende del
esqueleto de manejo de errores que esta slice especifica aquí por primera vez.

## What Changes

- Mapeo de errores síncronos de `POST /payments` (`409 INSUFFICIENT_FUNDS`, `409 WALLET_NOT_ACTIVE`,
  `400`, `404`, `413`, `429`, `503 PROVIDER_ERROR`) a un resultado terminal Fallido, antes o durante el
  envío del pago.
- Regla de "Expirado": si el *polling* de `GET /payments/{id}` no alcanza un estado terminal dentro de un
  tiempo máximo propio de la app, se declara Expirado (el tiempo máximo exacto queda como pregunta abierta
  en `design.md`, heredada de la HU original).
- Invocación del componente de presentación de resultado de **HU01-S6** con el estado (Fallido o Expirado)
  y los datos disponibles del pago (`id` si existe, fecha, destino, monto) al determinar cualquiera de los
  dos resultados; esta slice no describe la pantalla en sí, solo decide el estado y los datos que recibe.
- Distinción del mensaje de `409 INSUFFICIENT_FUNDS` frente a un error genérico, usando el catálogo de
  mensajes de **HU01-S0** (la etiqueta final de texto no está cerrada con producto, ver Open Questions).
- Entrega de la acción **Volver al inicio** (CA4) como la única acción ofrecida tras un resultado Fallido o
  Expirado; la acción **Reintentar** queda explícitamente en backlog (fuera de alcance).
- No hay cambios de contrato existente (no hay **BREAKING**): `POST /payments` y `GET /payments/{id}` ya
  son consumidos por HU01-S1 con el mismo contrato; esta slice solo especifica cómo la orquestación
  interpreta códigos de error y el agotamiento del *polling* que ese contrato ya contempla.

## Capabilities

### New Capabilities
<!-- Ninguna: esta slice no introduce una capability nueva. -->

### Modified Capabilities
- `payment-orchestration`: extiende la orquestación del pago (T7) construida en HU01-S1 con el mapeo de
  errores síncronos de `POST /payments` a un resultado Fallido y con la regla de "Expirado" por
  agotamiento del *polling* de `GET /payments/{id}`.

  **Nota de secuenciación:** `openspec list --specs` no muestra hoy ninguna capability archivada (la
  capability `payment-orchestration` solo existe como delta sin archivar en el change
  `hu01-s1-pago-exitoso`, y ya fue extendida por un segundo delta sin archivar en
  `hu01-s4-orquestador-pago`, que depende de la regla de "Expirado" que esta slice especifica por
  primera vez). Este delta se escribe como extensión directa del contrato de orquestación definido por
  HU01-S1, con `ADDED Requirements` (no `MODIFIED`) sobre el mismo criterio ya aplicado en HU01-S4, y
  asume que, al momento de aplicar/archivar este change, se resuelve el orden recomendado HU01-S1 →
  HU01-S3 → HU01-S4; ver Open Questions en `design.md`.

## Impact

- Código afectado: `web/` — la orquestación del pago (T7) construida en HU01-S1, específicamente el
  manejo de la respuesta de `POST /payments` y el ciclo de *polling* de `GET /payments/{id}`, más la
  invocación del componente de resultado de HU01-S6 con los estados Fallido/Expirado. Este proposal solo
  produce artefactos de planeación (specs, design, tasks); no modifica código todavía.
- APIs: no agrega ni modifica endpoints; interpreta códigos de error (`409/400/404/413/429/503`) de
  `POST /payments` y el agotamiento de tiempo del *polling* de `GET /payments/{id}`, ya contemplados en el
  contrato mock descrito en `HU01-S1-pago-exitoso.md`.
- Dependencias funcionales: depende de **HU01-S1** (reutiliza la base de T7 y el *polling* del camino
  feliz), de **HU01-S0** (catálogo de mensajes y componentes accesibles) y de **HU01-S6** (presentación del
  resultado Fallido/Expirado, incluida la acción "Volver al inicio"); es prerrequisito de **HU01-S4** (el
  manejo especial de `504` se apoya en el esqueleto de manejo de errores construido aquí), que ya existe
  como change de OpenSpec y queda referenciado como riesgo de secuenciación en `design.md`.
- Verificación: pruebas de caracterización de los escenarios de mock `REJECTED`, saldo insuficiente
  (`409 INSUFFICIENT_FUNDS`) y vencimiento del tiempo máximo de "Procesando" (resultado Expirado), por
  definir en `tasks.md`; meta de cobertura del código modificado ≥ 85%.
