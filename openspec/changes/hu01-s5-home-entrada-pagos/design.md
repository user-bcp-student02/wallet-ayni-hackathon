# Design

## Context

HU01-S0 ya define los componentes de UI accesibles y el catálogo de mensajes que esta slice reutiliza. Ver
`proposal.md` — Why para la motivación completa.

El repositorio actual (`web/src/app`) solo tiene `LoginComponent` y `DashboardComponent`
([dashboard.component.ts](../../../web/src/app/dashboard/dashboard.component.ts)); este último lista tareas
desde `GET /api/tasks` y no muestra saldo ni ofrece ninguna acción de pago. `api/app/main.py` solo expone
`/auth`, `/api/tasks`, `/api/summary`, `/api/requirements` y `/api/backend-health`: no existe hoy ningún
endpoint `/wallets/{id}`. Esta slice asume, como la HU original, que `GET /wallets/{id}` está disponible
(real o mock) con el contrato descrito en el Alcance de `HU01-S5-home-entrada-pagos.md`.

## Goals / Non-Goals

**Goals:**
- Especificar el contrato observable de la pantalla Home (capability `payment-home-entry`) — saludo, saldo
  disponible y la única acción "Nuevo pago" — de forma testeable e independiente de la implementación
  concreta (framework de UI, cliente HTTP).
- Dejar explícita la frontera entre esta slice y HU01-S0 (de la que consume componentes y catálogo) y
  HU01-S1 (a la que entrega el control al presionar "Nuevo pago"), para que ninguna de las dos necesite
  reabrir su contrato por causa de esta HU.

**Non-Goals:**
- No decide el framework ni la estructura de carpetas concreta de `web/` para la nueva pantalla Home (se
  resuelve en `tasks.md` al implementar).
- No define el contrato HTTP completo de `GET /wallets/{id}` (códigos de error, esquema JSON completo): la
  HU original no lo especifica tampoco; ver Open Questions.
- No decide dónde ni cómo se implementa `GET /wallets/{id}` (backend real en `api/` vs. mock en `web/`):
  ver Open Questions.
- No decide el comportamiento de Home ante un error al consultar `GET /wallets/{id}` (p. ej. falla de red):
  la HU original lo deja explícitamente pendiente de definición; ver Open Questions.

## Decisions

- **Una sola capability, no varias**: se agrupan saludo, saldo y la acción "Nuevo pago" en
  `payment-home-entry` en lugar de separarlas en capabilities independientes. Alternativa considerada:
  separar "presentación de saldo" de "navegación". Se descarta porque las tres piezas conforman una sola
  pantalla con un único objetivo (T1, "un objetivo por pantalla" según el catálogo de HU01-S0) y ninguna
  HU derivada necesita modificar una sin la otra.
- **Home no implementa la pantalla de selección de contacto**: `payment-home-entry` SHALL navegar hacia T2
  al presionar "Nuevo pago", pero no SHALL definir el contenido de esa pantalla — eso ya está especificado
  por HU01-S1 (`payment-request-capture`). Esto evita que esta slice reabra o duplique ese contrato.
- **El saldo se trata como dato de solo lectura obtenido por un único endpoint**: se especifica que Home
  SHALL consultar `GET /wallets/{id}` y mostrar el valor devuelto, sin prescribir el formato de
  presentación (moneda, decimales) ni el mecanismo de cacheo, que son decisiones de `tasks.md`.

## Risks / Trade-offs

- [El endpoint `GET /wallets/{id}` no existe en `api/` hoy] → Antes de iniciar `tasks.md`/implementación,
  confirmar con el equipo si esta slice se implementa contra un mock en `web/` o si primero se necesita un
  change separado en `api/` para el endpoint real; ver Open Questions. Mismo patrón ya seguido por HU01-S1
  con `/payments`.
- [El comportamiento ante error de `GET /wallets/{id}` no está definido] → No se infiere un comportamiento
  por adelantado (la HU original lo deja pendiente); se registra como Open Question y no se incluye en la
  spec hasta que se resuelva, para no inventar un requisito no aprobado.

## Open Questions

- ¿`GET /wallets/{id}` se implementa como mock dentro de `web/` para esta slice, o requiere primero un
  change en `api/` que lo exponga? No cambia la spec de esta slice (que es agnóstica a dónde vive el
  backend), pero sí determina el alcance de `tasks.md` cuando se apruebe la implementación.
- ¿Qué debe mostrar Home si falla la consulta de `GET /wallets/{id}` (p. ej. error de red o respuesta no
  exitosa)? La HU original no lo especifica y no se infiere un comportamiento; debe definirse con el
  equipo/negocio antes de implementar esta parte, y entonces se añadirá como requisito a la spec.
