# Design

## Context

HU01-S0 ya define los componentes de UI accesibles y el catálogo de mensajes que esta slice reutiliza.
HU01-S5 (Home) y HU01-S6 (pantalla de finalización) ya están especificadas como las HU que anteceden y
suceden a este flujo: HU01-S5 navega hacia T2 y HU01-S6 recibe el estado (Exitoso/Procesando) y los datos
del pago ya resueltos. Ver `proposal.md` — Why para la motivación completa.

El repositorio actual (`api/app/main.py`) solo expone `/auth` y `/tasks`; no existe hoy ningún endpoint
`/payments`. Esta slice asume, como la HU original, que `POST /payments` y `GET /payments/{id}` están
disponibles (reales o mock) con el contrato descrito en el Alcance de `HU01-S1-pago-exitoso.md`.

## Goals / Non-Goals

**Goals:**
- Especificar el contrato observable de las pantallas T2/T3/T4 (capability `payment-request-capture`) y de
  la orquestación T7 en su camino feliz (capability `payment-orchestration`), de forma testeable e
  independiente de la implementación concreta (framework de UI, cliente HTTP).
- Dejar explícita la frontera entre esta slice y sus dependencias (HU01-S0, HU01-S5, HU01-S6) para que
  HU01-S2/S3/S4 puedan extenderla sin reabrir su contrato.

**Non-Goals:**
- No decide el framework ni la estructura de carpetas concreta de `web/` para las nuevas pantallas (se
  resuelve en `tasks.md` al implementar).
- No define el contrato HTTP completo de `POST /payments` / `GET /payments/{id}` (códigos de error,
  esquema JSON completo): eso excede el camino feliz de esta slice y queda para HU01-S2/S3/S4, que sí
  cubren variantes de negocio y error.
- No decide dónde ni cómo se implementan `POST /payments` / `GET /payments/{id}` (backend real en `api/`
  vs. mock en `web/`): ver Open Questions.

## Decisions

- **Dos capabilities, no una**: se separa `payment-request-capture` (pantallas T2–T4, sin llamadas de red
  propias) de `payment-orchestration` (T7, integración con el backend). Alternativa considerada: una sola
  capability "payment-happy-path". Se descarta porque mezclaría requisitos de presentación —reutilizables
  tal cual por HU01-S2 (que solo agrega validación de rango sobre T3)— con requisitos de integración de
  red —reutilizados por HU01-S3/S4 con otras ramas de resultado—, dificultando que esas HU modifiquen solo
  la parte que les corresponde.
- **La pantalla de resultado no es parte de esta slice**: `payment-orchestration` SHALL invocar el
  componente de HU01-S6 con un estado y datos ya resueltos, pero no SHALL definir cómo se renderiza ese
  componente. Esto preserva la frontera ya establecida en HU01-S6 ("pantalla tonta respecto al negocio").
- **Idempotency-Key se trata como requisito de contrato, no de implementación**: se especifica su rango de
  longitud y unicidad por wallet (observable en la solicitud HTTP), sin prescribir el algoritmo de
  generación (UUID v4, hash, etc.), que es una decisión de `tasks.md`.

## Risks / Trade-offs

- [Los endpoints `/payments` no existen en `api/` hoy] → Antes de iniciar `tasks.md`/implementación,
  confirmar con el equipo si esta slice se implementa contra un mock en `web/` o si primero se necesita un
  change separado en `api/` para los endpoints reales; ver Open Questions.
- [Separar en dos capabilities agrega un archivo de spec adicional] → Mitigado porque ambas capabilities
  comparten frontera clara con las HU futuras (S2 solo toca `payment-request-capture`; S3/S4 solo tocan
  `payment-orchestration`), evitando reabrir ambas specs en cada slice derivada.
- [El polling de `GET /payments/{id}` no tiene intervalo ni timeout definidos en la HU original] →
  Mitigado parcialmente: el timeout especial `504 TIMEOUT_AFTER_PROCESSING` es explícitamente fuera de
  alcance de esta slice (HU01-S4); el intervalo de polling es un detalle de implementación sin impacto en
  el contrato observable y se define en `tasks.md`.

## Open Questions

- ¿`POST /payments` y `GET /payments/{id}` se implementan como mock dentro de `web/` para esta slice, o
  requieren primero un change en `api/` que los exponga? No cambia las specs de esta slice (que son
  agnósticas a dónde vive el backend), pero sí determina el alcance de `tasks.md` cuando se apruebe la
  implementación.
