# Tasks

Alcance: orquestación del pago en `web/` para HU01-S4 (capability `payment-orchestration`, delta de esta
slice). Requiere que el *polling* base de HU01-S1 y, preferiblemente, la regla de "Expirado" de HU01-S3
(ver `design.md` — Open Questions) ya estén implementados antes de iniciar.

## 1. Continuidad del polling tras un 504

- [ ] 1.1 Implementar el manejo de `504 TIMEOUT_AFTER_PROCESSING` en la respuesta de `POST /payments` para
      que no dispare un resultado de Fallido/Expirado inmediato y, en su lugar, inicie el *polling* de
      `GET /payments/{id}` igual que tras un `202 PENDING`. Verificar con una prueba de caracterización que
      confirme que un `504` en `POST /payments` no invoca el componente de resultado con Fallido/Expirado.
- [ ] 1.2 Implementar el manejo de `504 TIMEOUT_AFTER_PROCESSING` recibido durante una consulta de
      `GET /payments/{id}` en curso, de modo que el *polling* continúe (reintentando la consulta) en vez de
      detenerse. Verificar con una prueba que confirme que, tras un `504` intermedio, el *polling* emite al
      menos una consulta adicional a `GET /payments/{id}`.
- [ ] 1.3 Confirmar que, al alcanzar un estado terminal real (`APPROVED` o `REJECTED`) después de uno o más
      `504`, se invoca el componente de resultado de HU01-S6 con ese estado real y los datos del pago
      (`id`, fecha, destino, monto). Verificar con el escenario de mock `TIMEOUT_AFTER_PROCESSING`: el
      *polling* continúa tras el `504` y finalmente refleja el estado real, no un falso
      "Expirado"/"Fallido".

## 2. Reutilización de la regla de expiración existente

- [ ] 2.1 Confirmar que el *polling* continuado tras un `504` usa el mismo contador/temporizador de tiempo
      máximo ya implementado para "Expirado" (HU01-S3), sin un temporizador ni una condición de expiración
      adicionales específicos del caso `504`. Verificar con una prueba unitaria que confirme que el tiempo
      transcurrido antes del `504` cuenta para el mismo límite máximo.
- [ ] 2.2 Confirmar que, si el *polling* posterior a un `504` agota ese tiempo máximo sin resolver un
      estado terminal, se invoca el componente de resultado de HU01-S6 con el estado Expirado, con el mismo
      comportamiento que cualquier otro agotamiento de *polling*. Verificar con una prueba que simule el
      agotamiento del tiempo máximo tras un `504` y confirme la invocación con estado Expirado.

## 3. Mapeo de campos en respuestas posteriores a un 504

- [ ] 3.1 Verificar que el mapeo de campos camelCase/snake_case ya construido en HU01-S1 interpreta
      correctamente una respuesta de `GET /payments/{id}` en snake_case (`wallet_id`, `amount`, etc.)
      recibida durante el *polling* posterior a un `504`, sin necesidad de un mapeo adicional. Verificar
      con una prueba unitaria que ejercite el mapeo usando una respuesta de ejemplo tomada de este
      escenario.

## 4. Verificación end-to-end y contraste negativo

- [ ] 4.1 Ejecutar el escenario de mock `DELAYED_APPROVAL` (*polling* prolongado antes de resolver) de
      extremo a extremo y confirmar que, sin que medie un `504`, el comportamiento de HU01-S1 no se ve
      afectado por los cambios de esta slice. Verificar con la prueba de integración/e2e de este escenario.
- [ ] 4.2 Ejecutar el escenario de mock `TIMEOUT_AFTER_PROCESSING` de extremo a extremo y confirmar que el
      *polling* continúa tras el `504` y finalmente refleja el estado real (Exitoso o Fallido, según
      corresponda). Verificar con la prueba de integración/e2e de este escenario.
- [ ] 4.3 Ejecutar el escenario de mock `PROVIDER_ERROR` como contraste negativo y confirmar que se trata
      con la lógica de HU01-S3 (resultado Fallido inmediato), no con la lógica de continuación de esta
      slice — confirma la separación de responsabilidad entre ambos. Verificar con la prueba de
      integración/e2e de este escenario.
- [ ] 4.4 Confirmar que la cobertura del código modificado en `web/` para esta slice alcanza ≥ 85%.
      Verificar con el reporte de cobertura de `python tools/lab.py test web` ejecutado desde la raíz.
- [ ] 4.5 Registrar evidencia en el documento general de hallazgos (sección `web`, HU01-S4) con el
      resultado de las pruebas y la cobertura obtenida, conservando las secciones existentes del
      documento.

## Workflow follow-up

- Antes de iniciar la tarea 1.1, confirmar que HU01-S1 (polling base) y la regla de "Expirado" de HU01-S3
  ya están implementados y disponibles (como código, o al menos como change de OpenSpec aprobado); si no
  lo están, resolverlo primero (ver `design.md` — Open Questions).
- Archivar este change una vez aprobada la implementación, con revisión humana y pruebas reales
  ejecutadas (no inventadas), según el contrato general de agentes del repositorio. Se recomienda archivar
  en el orden HU01-S1 → HU01-S3 → HU01-S4 para que `payment-orchestration` quede consolidada
  correctamente en `openspec/specs/`.
