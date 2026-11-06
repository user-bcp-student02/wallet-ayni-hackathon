# Proposal

## Why

HU01 — Confirmación de pagos de bajo monto necesita un punto de entrada verificable por separado del
resto del flujo de pago: hoy **HU01-S1-pago-exitoso.md** ya depende de que exista una pantalla de Home que
muestre el saldo del pagador y lo dirija hacia "Nuevo pago" (T2), pero esa pantalla todavía no está
especificada como slice propia. **HU01-S5 — Home: entrada a la funcionalidad de pagos**
([HU01-S5-home-entrada-pagos.md](../../../HU01-S5-home-entrada-pagos.md)) aísla ese punto de entrada (T1)
del resto del flujo de pago — técnica de *slicing* por *workflow steps* — para poder construirlo, revisarlo
y desplegarlo como la puerta de acceso a la funcionalidad de pagos, independientemente de la orquestación
que viene después. Esta slice reemplaza el contenido de T1 que antes estaba duplicado en HU01-S1 (ver
"Origen y trazabilidad" en la HU).

Hoy `web/src/app` solo expone `LoginComponent` y `DashboardComponent`
([dashboard.component.ts](../../../web/src/app/dashboard/dashboard.component.ts)); este último lista tareas
desde `GET /api/tasks` y no muestra saldo ni ofrece ninguna acción de pago, por lo que no cumple el rol de
Home que exige esta HU.

## What Changes

- Nueva pantalla Home (T1) que, para un pagador autenticado, muestra el saludo "Bienvenido, {Usuario}" y el
  saldo disponible obtenido de `GET /wallets/{id}`.
- Una única acción principal y grande, **Nuevo pago**, que navega hacia la pantalla de selección de
  contacto (T2) cubierta por **HU01-S1-pago-exitoso.md**; Home no ofrece ninguna otra acción principal (el
  botón "Ver historial" permanece en backlog, fuera de alcance).
- La pantalla se construye sobre los componentes de UI y el catálogo de mensajes de **HU01-S0**, sin
  reimplementar texto, contraste o botones por su cuenta.
- No se modifica `LoginComponent`, `DashboardComponent` ni los endpoints hoy consumidos por el frontend
  (`/auth/login`, `/api/tasks`): el cambio es aditivo (nueva pantalla Home), no hay **BREAKING**.
- `GET /wallets/{id}` es un endpoint nuevo consumido por esta slice; no existe hoy en
  `api/app/main.py` (que solo expone `/auth`, `/api/tasks`, `/api/summary`, `/api/requirements` y
  `/api/backend-health`). Su disponibilidad (real o mock) es una dependencia externa que debe resolverse
  antes de implementar — queda registrada como pregunta abierta para `design.md`, igual que lo hizo
  HU01-S1 con `/payments`.

## Capabilities

### New Capabilities

- `payment-home-entry`: pantalla de entrada (Home) que saluda al pagador autenticado, muestra su saldo
  disponible (`GET /wallets/{id}`) y ofrece una única acción principal, **Nuevo pago**, que navega hacia la
  selección de contacto del flujo de pago (T1, consumiendo componentes y catálogo de HU01-S0).

### Modified Capabilities

<!-- Ninguna: `openspec list --specs` no encontró specs previas, y este cambio no modifica
     LoginComponent, DashboardComponent ni los contratos de /auth/login o /api/tasks. -->

## Impact

- Código afectado: `web/` — nueva pantalla/componente Home bajo `web/src/app/` (estructura concreta a
  definir en `design.md`); no se tocan `LoginComponent` ni `DashboardComponent`. Este proposal solo produce
  artefactos de planeación (specs, design, tasks); no modifica código todavía.
- APIs: consume `GET /wallets/{id}` para el saldo disponible. Este endpoint no existe hoy en
  `api/app/main.py`; su implementación (real o mock) es una dependencia externa a esta slice y debe
  resolverse antes de poder implementarla — queda como pregunta abierta en `design.md`.
- Dependencias funcionales: requiere los componentes y catálogo de mensajes de **HU01-S0**; es
  prerrequisito de **HU01-S1** (la navegación de "Nuevo pago" entrega el control a T2, cubierta por esa
  HU). Ninguna de esas dos HU se modifica en este change.
- Verificación: pruebas de caracterización de la carga de Home con saldo disponible (camino feliz) y de la
  navegación desde "Nuevo pago" hacia la selección de contacto, por definir en `tasks.md`; meta de
  cobertura del código modificado ≥ 85%. El comportamiento de Home ante un error al consultar
  `GET /wallets/{id}` queda pendiente de definición (ver "Preguntas abiertas" en la HU original) y se
  trata como pregunta abierta en `design.md`.
