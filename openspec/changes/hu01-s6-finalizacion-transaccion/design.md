# Design

## Context

HU01-S0 ya define los componentes de UI accesibles y el catálogo de mensajes que este componente reutiliza
para sus textos. HU01-S5 (Home) ya especifica el destino al que navega la acción "Volver al inicio".
HU01-S1, HU01-S3 y HU01-S4 ya invocan "el componente de HU01-S6" asumiendo su existencia, sin haberlo
especificado formalmente hasta este change: esta slice formaliza ese contrato sin modificar los requisitos
ya descritos en esas tres HU (que siguen decidiendo el estado y los datos, no la presentación). Ver
`proposal.md` — Why para la motivación completa.

`web/src/app` hoy solo tiene `LoginComponent` y `DashboardComponent`; no existe todavía ningún componente
del flujo de pago (T1–T4, T7) ni, por lo tanto, el punto de invocación real de este componente. Ese punto
de invocación se construye en HU01-S1/S3/S4, no en esta slice.

## Goals / Non-Goals

**Goals:**
- Especificar el contrato observable del componente de presentación de resultado (capability
  `payment-result-presentation`) de forma testeable e independiente de la implementación concreta
  (framework de UI), para que HU01-S1, HU01-S3 y HU01-S4 puedan invocarlo sin ambigüedad sobre qué recibe
  y qué muestra.
- Dejar explícita la frontera "pantalla tonta": el componente no decide el estado ni interpreta respuestas
  del backend, solo presenta lo que recibe.

**Non-Goals:**
- No decide el framework ni la estructura de carpetas concreta de `web/` para este componente (se resuelve
  en `tasks.md` al implementar).
- No define el punto de invocación real (la orquestación de HU01-S1/S4 o el mapeo de errores de HU01-S3):
  eso ya está (o estará) especificado en esas HU; aquí solo se consume desde su lado como un componente
  con una interfaz de entrada fija (estado + datos).
- No define el contenido final de copy de los mensajes (catálogo de HU01-S0, pendiente de cierre con
  producto/diseño): se usa el mecanismo de catálogo tal como ya existe.
- No decide ni cambia la regla de cuánto tiempo transcurre en "Procesando" antes de declarar "Expirado":
  esa decisión es de quien invoca el componente (HU01-S3), no de este.

## Decisions

- **Un solo componente para los tres estados terminales y "Procesando"**: se especifica como una única
  capability (`payment-result-presentation`) con un requisito por estado, en vez de cuatro componentes
  separados. Alternativa considerada: un componente por estado. Se descarta porque la HU original exige
  explícitamente estructura visual consistente entre estados (mismos cuatro campos en los tres estados
  terminales) y reutilización por tres HU distintas (S1, S3, S4); cuatro componentes duplicarían ese
  layout común y el riesgo de inconsistencia que la HU busca evitar.
- **La interfaz de entrada es un estado + datos ya resueltos, no un código HTTP**: el requisito de frontera
  ("no realiza llamadas propias ni mapea códigos de error") se especifica como comportamiento observable
  (ausencia de llamadas HTTP propias), no como una restricción de implementación. Esto permite que
  HU01-S1/S3/S4 cambien su propia lógica de mapeo de errores sin reabrir esta spec, siempre que sigan
  entregando el mismo contrato de entrada (estado + `id`/fecha/destino/monto).
- **La persistencia del estado "Procesando" al cerrar/reabrir la app es un requisito de este componente,
  no del orquestador**: se eligió así porque la HU la describe como parte de lo que debe "no perderse" al
  reabrir, que es una propiedad observable de la pantalla (qué se muestra al reabrir), no de cómo el
  orquestador reanuda el polling (eso es decisión de HU01-S1/S3/S4). El mecanismo concreto de persistencia
  (almacenamiento local, reconstrucción desde el último estado conocido) se decide en `tasks.md`.

## Risks / Trade-offs

- [HU01-S1, HU01-S3 y HU01-S4 ya fueron redactadas asumiendo este componente antes de que existiera su
  spec] → Mitigado: este change no modifica los requisitos de esas tres HU (no hay "Modified
  Capabilities"); solo formaliza el contrato que ya asumían, con el mismo contrato de datos (`id`, fecha,
  destino, monto) que ellas ya describen.
- [La regla de "qué cuenta como el mismo intento" al cerrar y reabrir la app no está completamente
  definida a nivel de implementación] → Mitigado parcialmente: el requisito observable (no perder el
  intento visible) ya es suficiente para especificar y probar el comportamiento esperado; el mecanismo
  concreto se resuelve en `tasks.md` sin impacto en esta spec.
- [Cuatro estados en un solo componente podría complicar las pruebas si se implementan como una sola rama
  de código muy anidada] → Mitigado porque la spec exige un requisito y escenario por estado de forma
  independiente, lo que permite (y fomenta) implementarlos como casos testeables por separado aunque
  compartan el mismo componente visual.
