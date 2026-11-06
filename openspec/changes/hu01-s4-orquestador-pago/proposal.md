# Proposal

## Why

HU01-S1 ya implementa el *polling* de `GET /payments/{id}` para el camino feliz, y HU01-S3 define el
esqueleto de manejo de errores y la regla de "Expirado". Pero ninguna de las dos cubre el caso
**`504 TIMEOUT_AFTER_PROCESSING`**: un timeout del backend que puede ocurrir *después* de que el proveedor
ya aprobó el pago. Si la app tratara ese `504` como cualquier otro error (regla simétrica de HU01-S3),
mostraría "Fallido"/"Expirado" a un pagador cuyo dinero sí se movió, generando pérdida de confianza y
posible doble pago si el usuario reintenta creyendo que falló. **HU01-S4 — Caso especial 504: timeout tras
procesamiento ya aprobado** ([HU01-S4-orquestador-pago.md](../../../HU01-S4-orquestador-pago.md)) aísla
esta regla asimétrica para implementarla, revisarla y probarla sin mezclarla con el resto del manejo de
errores.

## What Changes

- Manejo especial de `504 TIMEOUT_AFTER_PROCESSING` recibido en `POST /payments` o en `GET /payments/{id}`
  durante el *polling*: la app **no** muestra "Expirado" ni "Fallido" de inmediato por ese código.
- Continuación del *polling* de `GET /payments/{id}` tras un `504`, hasta alcanzar un estado terminal real
  (`APPROVED` o `REJECTED`), respetando el mismo tiempo máximo de la app ya definido por HU01-S3 para esa
  continuación (sin introducir una regla de expiración distinta ni un tiempo máximo adicional).
- Si el *polling* posterior a un `504` agota ese tiempo máximo sin resolver, se aplica la misma regla de
  "Expirado" de HU01-S3, tal cual, como cualquier otro agotamiento de *polling*.
- Verificación explícita de que el mapeo de campos camelCase/snake_case construido en HU01-S1 interpreta
  correctamente las respuestas de `GET /payments/{id}` recibidas tras un `504` (no se introduce un mapeo
  nuevo, se confirma que el existente cubre también este flujo).
- No hay cambios de contrato existente (no hay **BREAKING**): `POST /payments` y `GET /payments/{id}`
  conservan su contrato; este change solo especifica cómo la orquestación interpreta un código de
  respuesta (`504`) que el contrato ya contempla.

## Capabilities

### New Capabilities
<!-- Ninguna: esta slice no introduce una capability nueva. -->

### Modified Capabilities
- `payment-orchestration`: extiende el *polling* de `GET /payments/{id}` para que un `504
  TIMEOUT_AFTER_PROCESSING` (en `POST /payments` o durante el *polling*) no se trate como resultado
  terminal, sino que el *polling* continúe hasta un estado terminal real, reutilizando la regla de
  "Expirado" existente si ese *polling* agota su tiempo máximo.

  **Nota de secuenciación:** `openspec list --specs` no muestra hoy ninguna capability archivada (la
  capability `payment-orchestration` solo existe como delta sin archivar en el change
  `hu01-s1-pago-exitoso`, y la regla de "Expirado" que se reutiliza aquí está descrita en
  `HU01-S3-resultados-no-exitosos.md` pero **HU01-S3 todavía no tiene un change de OpenSpec propio**). Este
  delta se escribe como extensión directa del requisito de *polling* definido por HU01-S1 y asume que, al
  momento de aplicar/archivar este change, HU01-S1 (y preferiblemente HU01-S3, una vez tenga su propio
  change) ya estarán archivados en ese orden; ver Open Questions en `design.md`.

## Impact

- Código afectado: `web/` — la orquestación del pago (T7) construida en HU01-S1, específicamente el
  manejo de la respuesta de `POST /payments` y el ciclo de *polling* de `GET /payments/{id}`. Este proposal
  solo produce artefactos de planeación (specs, design, tasks); no modifica código todavía.
- APIs: no agrega ni modifica endpoints; interpreta un código de respuesta (`504
  TIMEOUT_AFTER_PROCESSING`) ya contemplado en el contrato mock de `POST /payments` y
  `GET /payments/{id}` descrito en `HU01-S1-pago-exitoso.md` y `HU01-S4-orquestador-pago.md`.
- Dependencias funcionales: depende de HU01-S1 (reutiliza el *polling* base de T7) y de HU01-S3 (reutiliza
  el esqueleto de manejo de errores/resultados terminales y la regla de tiempo máximo para "Expirado");
  HU01-S3 no tiene todavía un change de OpenSpec propio, lo cual queda registrado como riesgo/pregunta
  abierta en `design.md`. No tiene dependientes: es la última slice del backlog derivado de HU01.
- Verificación: pruebas de caracterización de los escenarios de mock `DELAYED_APPROVAL` y
  `TIMEOUT_AFTER_PROCESSING` (continuación correcta del *polling* tras el `504`), y del escenario
  `PROVIDER_ERROR` como contraste negativo (debe seguir tratándose como Fallido, no con esta lógica de
  continuación), por definir en `tasks.md`; meta de cobertura del código modificado ≥ 85%.
