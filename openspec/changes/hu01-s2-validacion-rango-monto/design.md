# Design

## Context

HU01-S1 ya construye T3/T4 y la orquestación `POST /payments` en su camino feliz (sin validar rango). Esta
slice añade la regla de validación de monto sobre esas mismas pantallas y esa misma llamada, sin modificar
su contrato. Ver `proposal.md` — Why para la motivación completa.

Como en HU01-S1, `api/app/main.py` hoy solo expone `/auth` y `/tasks`; `POST /payments` sigue sin existir
como endpoint real (es una dependencia externa a esta slice, igual que para HU01-S1). La validación de
rango del lado servidor descrita aquí se especifica como comportamiento observable de ese endpoint, sea
cual sea su implementación final (real en `api/` o mock en `web/`).

## Goals / Non-Goals

**Goals:**
- Especificar el contrato observable de la validación de rango de monto (S/ 10–S/ 1000) en la app (antes
  de avanzar de T3 a T4) y en el servidor (al orquestar `POST /payments`), de forma testeable e
  independiente del framework de UI o del cliente HTTP.
- Dejar explícito que el piso técnico de captura (S/ 0.01) es una regla de entrada separada de la regla de
  negocio de rango, para que ambas puedan verificarse de forma independiente.
- Garantizar que una discrepancia cliente-servidor (monto que pasa la validación de la app pero es
  rechazado por el servidor) resuelve en el mismo mensaje de bloqueo, sin estado "Procesando" falso.

**Non-Goals:**
- No decide el mecanismo concreto de validación (validador síncrono del formulario, framework de UI,
  librería de parsing numérico): se resuelve en `tasks.md` al implementar.
- No redefine el contrato HTTP completo de `POST /payments` (no es **BREAKING**): solo añade la condición
  de rechazo por rango sobre un endpoint ya definido en HU01-S1.
- No decide dónde vive la validación de servidor (backend real en `api/` vs. mock en `web/`): hereda la
  misma pregunta abierta que HU01-S1 sobre la ubicación de `POST /payments`.

## Decisions

- **Una sola capability, no dos**: se agrupan la validación de cliente y la de servidor bajo
  `payment-amount-range-validation` en lugar de separarlas en dos capabilities. Alternativa considerada:
  separar `payment-amount-range-validation` (cliente) de una capability de validación de servidor.
  Se descarta porque ambas mitades comparten la misma regla de negocio (S/ 10–S/ 1000) y el mismo mensaje
  de bloqueo; dividirlas duplicaría la descripción del rango en dos specs sin ganar una frontera de
  implementación útil (a diferencia de HU01-S1, donde `payment-request-capture` y `payment-orchestration`
  sí tienen ciclos de vida de implementación distintos).
- **El piso técnico de captura es un requisito propio, no una variante del rango de negocio**: se
  especifica como requirement independiente porque aplica siempre (incluso si en el futuro cambiara el
  mínimo de negocio de S/ 10), y porque actúa a nivel de entrada del campo, no como bloqueo posterior al
  intentar continuar.
- **El mensaje de bloqueo es una referencia al catálogo de HU01-S0, no un texto propio**: evita introducir
  una segunda fuente de copy para el mismo tipo de mensaje de error, consistente con la decisión ya tomada
  en HU01-S0 de centralizar todo texto visible al pagador.
- **No se crea un delta sobre `payment-request-capture` ni `payment-orchestration`**: esas capabilities de
  HU01-S1 siguen sin sincronizarse a `openspec/specs/` (ver Modified Capabilities en `proposal.md`), así
  que no hay un path existente que modificar hoy. Cuando HU01-S1 se archive y sincronice, esta validación
  podría reconsiderarse como un delta MODIFIED sobre esas capabilities en lugar de una capability nueva;
  se deja como nota para quien implemente, no como tarea de esta slice.

## Risks / Trade-offs

- [`POST /payments` no existe aún como endpoint real en `api/`] → Mismo riesgo heredado de HU01-S1: antes
  de implementar la validación de servidor, confirmar si se construye contra un mock en `web/` o si
  requiere primero el endpoint real en `api/`.
- [Duplicar la regla S/ 10–S/ 1000 en cliente y servidor puede desincronizarse] → Mitigado especificando
  ambos lados como el mismo rango textual en una sola spec, de modo que cualquier cambio futuro del rango
  obligue a actualizar un único archivo de requirements y ambas implementaciones deriven de él.
- [Esta capability no se fusiona con las de HU01-S1 porque esas aún no están en `openspec/specs/`] →
  Mitigado documentando explícitamente la intención de reconsiderar la fusión una vez HU01-S1 se archive;
  no bloquea la implementación de esta slice mientras tanto.
