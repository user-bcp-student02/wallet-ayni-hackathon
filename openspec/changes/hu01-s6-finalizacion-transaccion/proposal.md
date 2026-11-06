# Proposal

## Why

HU01-S1 (camino feliz), HU01-S3 (resultados no exitosos) y HU01-S4 (caso especial `504`) ya invocan "el
componente de HU01-S6" para presentar el estado terminal (Exitoso, Fallido, Expirado) o el intermedio
"Procesando" de un pago, pero esa pantalla de finalización (T5) nunca se especificó como una capability
propia: HU01-S1 la asumió existente al describir su invocación. Sin esta slice, las tres HU que dependen
de ella no tienen un contrato verificable de qué datos recibe la pantalla, qué muestra en cada estado ni
cómo navega de vuelta a Home, y cada una podría terminar reimplementando su propia variante de la pantalla
de resultado, duplicando lógica y arriesgando inconsistencia entre estados. **HU01-S6 — Pantalla de
finalización de transacción de pago** ([HU01-S6-finalizacion-transaccion.md](../../../HU01-S6-finalizacion-transaccion.md))
cierra ese vacío: especifica un único componente de presentación, "tonto" respecto al negocio, que recibe
un estado y datos ya resueltos por quien lo invoca y los muestra de forma consistente.

## What Changes

- Nuevo componente de presentación de resultado (T5) que renderiza, con la misma estructura visual, los
  tres estados terminales — **Exitoso**, **Fallido**, **Expirado** — y el estado intermedio
  **"Procesando"**.
- En los tres estados terminales se muestran siempre los mismos campos: referencia (el `id` del pago),
  fecha, destino y monto, junto con el mensaje correspondiente del catálogo de mensajes de HU01-S0.
- En el estado "Procesando" se muestra un mensaje tranquilizador en lugar de los datos finales, y el
  intento no se pierde si el pagador cierra y reabre la app.
- Acción **Volver al inicio**, disponible en cualquier estado terminal mostrado, que navega a Home
  (HU01-S5).
- El componente es puramente de presentación: no realiza llamadas propias a `POST /payments` ni a
  `GET /payments/{id}`, ni mapea códigos de error HTTP; recibe el estado y los datos ya resueltos por quien
  lo invoca (HU01-S1, HU01-S3 o HU01-S4).
- No hay cambios de contrato existente (no hay **BREAKING**): este change solo produce los artefactos de
  planeación (specs, design, tasks) para una capability nueva, consumida por otras slices que ya asumían
  su existencia sin haberla especificado formalmente.

## Capabilities

### New Capabilities

- `payment-result-presentation`: componente de presentación único que muestra el estado intermedio
  "Procesando" o uno de los tres estados terminales (Exitoso, Fallido, Expirado) de un pago, con sus datos
  (`id`, fecha, destino, monto) ya resueltos por quien lo invoca, y ofrece la acción "Volver al inicio"
  hacia Home.

### Modified Capabilities

<!-- Ninguna: openspec/specs/ no contiene specs previas (`openspec list --specs` no encontró specs), y
     `payment-request-capture` / `payment-orchestration` (definidas en el change hu01-s1-pago-exitoso, aún
     sin archivar) ya asumen la invocación a este componente sin requerir cambios de requisitos sobre
     ellas mismas. -->

## Impact

- Código afectado: `web/` — nuevo componente de presentación de resultado bajo `web/src/app/` (estructura
  concreta a definir en `design.md`). No modifica `LoginComponent`, `DashboardComponent` ni los componentes
  de HU01-S0. Este proposal solo produce los artefactos de planeación; no modifica código todavía.
- APIs: ninguna; el componente no realiza llamadas HTTP propias (ni `POST /payments` ni
  `GET /payments/{id}`), consistente con el Alcance de la HU.
- Dependencias funcionales: requiere los componentes de UI y el catálogo de mensajes de **HU01-S0**, y el
  destino de navegación de **HU01-S5** (Home). Es un componente proveído para **HU01-S1**, **HU01-S3** y
  **HU01-S4**, que deciden el estado y los datos pero no describen la pantalla por su cuenta.
- Verificación: pruebas de renderizado del componente para cada estado (Procesando, Exitoso, Fallido,
  Expirado), persistencia del intento al cerrar/reabrir la app durante "Procesando", y navegación de
  "Volver al inicio" hacia Home, por definir en `tasks.md`; meta de cobertura del código modificado ≥ 85%.
