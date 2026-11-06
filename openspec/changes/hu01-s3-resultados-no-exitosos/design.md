# Design

## Context

HU01-S1 ya construye la orquestación del pago (T7) y el *polling* base de `GET /payments/{id}` hasta
`APPROVED`, dentro de la capability `payment-orchestration`. **HU01-S4 — Caso especial 504** ya existe
como change de OpenSpec y agrega, sobre esa misma capability, la continuación del *polling* tras un
`504 TIMEOUT_AFTER_PROCESSING` reutilizando — por referencia, sin re-especificarla — la regla de
"Expirado" que esta slice (HU01-S3) define aquí por primera vez. Ver `proposal.md` — Why y la nota de
secuenciación en Capabilities para el detalle de esa dependencia invertida en el tiempo.

El repositorio actual (`api/app/main.py`) solo expone `/auth` y `/tasks`; no existe hoy ningún endpoint
`/payments`, real o mock. Esta slice asume, como HU01-S1 y HU01-S4, que `POST /payments` y
`GET /payments/{id}` están disponibles con el contrato descrito en el Alcance de
`HU01-S1-pago-exitoso.md` y `HU01-S3-resultados-no-exitosos.md`.

## Goals / Non-Goals

**Goals:**
- Especificar el contrato observable del mapeo de errores síncronos de `POST /payments` y de la regla de
  "Expirado" por agotamiento del *polling* (capability `payment-orchestration`, ver `specs/`), de forma
  testeable e independiente de la implementación concreta (cliente HTTP, framework de UI).
- Dejar explícita la frontera con HU01-S6: esta slice decide el estado (Fallido/Expirado) y los datos del
  pago, pero no describe cómo se renderiza la pantalla de resultado ni la acción "Volver al inicio" en sí
  misma (eso es responsabilidad de HU01-S6).
- Dejar una base de errores/resultados terminales estable para que HU01-S4 (ya existente) pueda seguir
  reutilizando la regla de "Expirado" sin reabrir el contrato de esta slice.

**Non-Goals:**
- No define el manejo del `504 TIMEOUT_AFTER_PROCESSING`: ese caso asimétrico es responsabilidad exclusiva
  de HU01-S4, que ya lo especifica reutilizando la regla de "Expirado" definida aquí.
- No decide el tiempo máximo exacto de "Procesando" antes de declarar "Expirado", ni la etiqueta final de
  texto de los estados terminales: ambos quedan como preguntas abiertas heredadas de la HU original (ver
  Open Questions), pendientes de validarse con producto antes de implementar.
- No especifica la acción **Reintentar** ni la reutilización de `Idempotency-Key` entre intentos: quedan
  explícitamente en backlog, fuera de alcance de esta slice.
- No decide el intervalo de *polling*, el mecanismo de backoff ni la estructura de carpetas concreta de
  `web/`: se resuelve en `tasks.md` al implementar.

## Decisions

- **Un solo delta ADDED sobre `payment-orchestration`, no una capability nueva**: los cuatro requisitos de
  esta slice (mapeo de errores síncronos, regla de Expirado, acción única, mensaje distintivo de saldo
  insuficiente) se agregan como `ADDED Requirements` sobre la misma capability `payment-orchestration` que
  HU01-S1 empezó y HU01-S4 ya extendió. Alternativa considerada: crear una capability separada
  `payment-error-handling`. Se descarta porque el mapeo de errores y la regla de Expirado son, en esencia,
  ramas adicionales del mismo contrato de orquestación (qué estado terminal resulta de `POST /payments` /
  `GET /payments/{id}`), no una preocupación distinta; separarla obligaría a HU01-S4 a declarar una
  dependencia cruzada entre dos capabilities en vez de una sola.
- **La regla de Expirado se especifica aquí, no se asume desde HU01-S4**: aunque HU01-S4 ya la referencia,
  este change es el que la define por primera vez como requisito propio, cerrando la brecha de
  secuenciación documentada en el `design.md` de HU01-S4 ("HU01-S3 todavía no tiene un change de OpenSpec
  propio"). No se re-escribe el delta de HU01-S4 como parte de este change: su referencia por nombre a
  "la misma regla ya existente" sigue siendo válida una vez archivado este change en el orden correcto.
- **El mensaje distintivo de saldo insuficiente es un requisito propio, no un detalle de UI**: se declara
  como requisito explícito (en vez de dejarlo implícito en el catálogo de mensajes de HU01-S0) porque el
  CA4 original lo exige como criterio observable y probable, aunque la etiqueta final de texto no esté
  cerrada con producto.

## Risks / Trade-offs

- [El orden de archivado recomendado es HU01-S1 → HU01-S3 → HU01-S4, pero HU01-S4 ya existe como change
  sin que HU01-S3 estuviera archivado] → Mitigado: el delta de HU01-S4 usa `ADDED Requirements` sobre
  conceptos nuevos (continuación de *polling* tras `504`) y referencia la regla de Expirado por nombre sin
  re-especificarla, por lo que archivar este change después no debería requerir cambios en HU01-S4; aun
  así, antes de archivar cualquiera de los dos, confirmar con `openspec validate` que ambos deltas siguen
  siendo consistentes contra el spec canónico resultante.
- [El tiempo máximo exacto de "Procesando" no está definido] → Mitigado parcialmente: el requisito de
  "Expirado" se especifica en términos de "un tiempo máximo propio de la app" sin fijar un valor, dejando
  la decisión concreta para `tasks.md` una vez que producto confirme el valor (ver Open Questions); el
  contrato observable (estado Expirado tras agotar ese tiempo) no cambia si el valor cambia después.
- [Los endpoints `/payments` no existen en `api/` hoy] → Mismo riesgo ya registrado en HU01-S1 y HU01-S4:
  antes de iniciar la implementación, confirmar si esta slice se implementa contra un mock en `web/` o si
  se necesita un change separado en `api/` para los endpoints reales.

## Open Questions

- ¿Cuál es el tiempo máximo exacto que debe transcurrir en "Procesando" antes de declarar "Expirado"? No
  cambia el contrato de esta slice (que lo trata como un parámetro configurable de la app), pero sí
  determina el valor concreto a implementar en `tasks.md`.
- ¿Cuáles son las etiquetas finales de texto para los estados Fallido y Expirado, y el contenido
  definitivo del catálogo de mensajes para el caso `409 INSUFFICIENT_FUNDS`? Pendiente de cierre con
  producto; mientras tanto, `tasks.md` puede usar un texto provisional sin bloquear la implementación del
  contrato.
