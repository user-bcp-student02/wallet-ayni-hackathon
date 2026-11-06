# Design

## Context

HU01-S1 ya construye el *polling* base de `GET /payments/{id}` (camino feliz, hasta `APPROVED`) y define el
mapeo de campos camelCase/snake_case, ambos dentro de la capability `payment-orchestration`. HU01-S3
extiende esa misma capability con el mapeo de errores síncronos y la regla de "Expirado" por agotamiento
de tiempo máximo — pero, a diferencia de HU01-S1, **HU01-S3 todavía no tiene un change de OpenSpec propio**
(solo existe como documento `HU01-S3-resultados-no-exitosos.md`). Ver `proposal.md` — Why y la nota de
secuenciación en Capabilities para el detalle de esta brecha.

El caso `504 TIMEOUT_AFTER_PROCESSING` es asimétrico respecto a los demás errores de HU01-S3: el backend
puede haber aprobado el pago aunque la respuesta HTTP inmediata sea un error de timeout. Tratarlo con la
misma lógica simétrica de HU01-S3 (error ⇒ Fallido) arriesga mostrar un resultado incorrecto a un pagador
cuyo dinero sí se movió.

## Goals / Non-Goals

**Goals:**
- Especificar el contrato observable de la continuación del *polling* tras un `504` recibido en
  `POST /payments` o en `GET /payments/{id}` (capability `payment-orchestration`, ver `specs/`), de forma
  testeable e independiente de la implementación concreta (cliente HTTP, framework de UI).
- Dejar explícito que esta slice **no** introduce una regla de expiración distinta: reutiliza, sin
  modificarla, la que define HU01-S3 para cualquier agotamiento de *polling*.
- Documentar la brecha de secuenciación (HU01-S3 sin change propio) como riesgo explícito en lugar de
  asumir en silencio que ya existe.

**Non-Goals:**
- No define el mapeo de errores síncronos (`409/400/404/413/429/503`) ni la regla de "Expirado" en sí
  misma: esos son responsabilidad de HU01-S3, no de esta slice.
- No decide el intervalo de *polling*, el mecanismo de backoff ni la estructura de carpetas concreta de
  `web/` para implementar la continuación tras un `504`: se resuelve en `tasks.md` al implementar.
- No modifica el contrato HTTP de `POST /payments` / `GET /payments/{id}`: `504
  TIMEOUT_AFTER_PROCESSING` ya existe en ese contrato (mock); esta slice solo especifica cómo la
  orquestación lo interpreta.

## Decisions

- **Requisitos nuevos (ADDED), no modificación del requisito de polling existente**: el requisito
  "Polling hasta estado terminal aprobado" de HU01-S1 describe el camino feliz (`202 PENDING` →
  `APPROVED`) y no necesita reescribirse; el comportamiento ante un `504` es una preocupación adicional
  que no cambia ese contrato existente. Alternativa considerada: reescribir ese requisito como
  `MODIFIED` incorporando el caso `504`. Se descarta porque mezclaría dos contratos con dueños distintos
  (HU01-S1 para el camino feliz, HU01-S4 para el caso especial de timeout) y dificultaría rastrear qué
  HU introdujo cada regla.
- **La regla de expiración se referencia, no se re-especifica**: el requisito "Reutilización de la regla
  de expiración existente tras un 504" declara explícitamente que se aplica "la misma regla ya
  existente" sin repetir sus condiciones (tiempo máximo exacto, etc.), para que un cambio futuro en esa
  regla (propiedad de HU01-S3) no deje desactualizado este delta.
- **El mapeo de campos se verifica, no se redefine**: se agrega un requisito explícito que cubre el caso
  "respuesta snake_case tras un 504", en vez de asumir que el mapeo genérico de HU01-S1 ya lo cubre
  implícitamente — así queda como contrato probado, no como supuesto.

## Risks / Trade-offs

- [HU01-S3 no tiene todavía un change de OpenSpec propio, pero este delta depende de su regla de
  "Expirado"] → Mitigado parcialmente: el requisito de esta slice reutiliza esa regla por referencia
  (sin re-especificarla), por lo que no se rompe si la regla de HU01-S3 cambia antes de implementarse;
  sin embargo, antes de implementar esta slice debe existir (al menos como documento aprobado o como
  change de OpenSpec) la definición concreta del tiempo máximo de "Expirado". Ver Open Questions.
- [La capability `payment-orchestration` no está archivada en `openspec/specs/`, solo existe como delta en
  `hu01-s1-pago-exitoso`] → Si se archiva `hu01-s1-pago-exitoso` antes que este change, `openspec list
  --specs` empezará a mostrar `payment-orchestration` como capability real; en ese momento este delta
  debería revisarse para confirmar que sigue aplicando correctamente contra el spec archivado (ver
  `openspec-update-change`), aunque su contenido no debería requerir cambios porque usa `ADDED
  Requirements` sobre conceptos nuevos, no `MODIFIED` sobre texto existente.
- [Probar el caso `504` requiere que el mock de backend soporte el escenario `TIMEOUT_AFTER_PROCESSING`] →
  Mitigado: la HU original ya lista ese escenario de mock junto con `DELAYED_APPROVAL` y `PROVIDER_ERROR`
  como contraste negativo; se usan tal cual en `tasks.md`.

## Open Questions

- ¿Se crea un change de OpenSpec propio para HU01-S3 antes de implementar HU01-S4, o se asume su regla de
  "Expirado" directamente desde `HU01-S3-resultados-no-exitosos.md`? No cambia las specs de esta slice
  (que reutilizan esa regla por referencia), pero sí determina el orden de archivado recomendado
  (HU01-S1 → HU01-S3 → HU01-S4) antes de iniciar la implementación de `tasks.md`.
