# Tasks

Alcance: orquestación del pago en `web/` para HU01-S3 (capability `payment-orchestration`, delta de esta
slice). Requiere que el camino feliz y el *polling* base de HU01-S1 ya estén implementados antes de
iniciar.

## 1. Mapeo de errores síncronos de POST /payments

- [ ] 1.1 Implementar el mapeo de los errores `400`, `404`, `413` y `429` de `POST /payments` al estado
      Fallido, invocando el componente de resultado de HU01-S6 con ese estado y los datos disponibles del
      pago (`id` si existe, fecha, destino, monto). Verificar con una prueba de caracterización por cada
      código que confirme la invocación con estado Fallido.
- [ ] 1.2 Implementar el mapeo de `409 WALLET_NOT_ACTIVE` al estado Fallido, con el mismo criterio que el
      resto de errores genéricos. Verificar con una prueba que confirme la invocación con estado Fallido
      para este código.
- [ ] 1.3 Implementar el mapeo de `503 PROVIDER_ERROR` al estado Fallido, confirmando que no se inicia ni
      continúa el *polling* de `GET /payments/{id}` tras este error (el backend no creó operación ni
      reserva). Verificar con una prueba que confirme la invocación con estado Fallido y la ausencia de
      llamadas a `GET /payments/{id}` tras este código.
- [ ] 1.4 Implementar el mapeo de `409 INSUFFICIENT_FUNDS` al estado Fallido usando un mensaje del
      catálogo de mensajes de HU01-S0 que distinga este caso de un error genérico. Verificar con una
      prueba que confirme que el mensaje mostrado para este código difiere del mensaje genérico de
      Fallido.

## 2. Resultado Expirado por agotamiento del polling

- [ ] 2.1 Implementar el temporizador de tiempo máximo de "Procesando" para el *polling* de
      `GET /payments/{id}`, usando un valor configurable (placeholder hasta que producto confirme el valor
      final, ver `design.md` — Open Questions). Verificar con una prueba unitaria que confirme que el
      temporizador se inicia junto con el primer *polling* y se detiene al alcanzar un estado terminal.
- [ ] 2.2 Implementar la invocación del componente de resultado de HU01-S6 con el estado Expirado y los
      datos disponibles del pago (`id` si existe, fecha, destino, monto) cuando el temporizador del paso
      2.1 se agota sin un estado terminal. Verificar con una prueba que simule el agotamiento del tiempo
      máximo y confirme la invocación con estado Expirado.

## 3. Acción única tras un resultado no exitoso

- [ ] 3.1 Confirmar que, al invocar el componente de resultado de HU01-S6 con estado Fallido o Expirado,
      solo se habilita la acción "Volver al inicio" y no se ofrece una acción "Reintentar" (queda en
      backlog). Verificar con una prueba de componente que confirme la ausencia del botón "Reintentar" en
      ambos estados.

## 4. Verificación end-to-end

- [ ] 4.1 Ejecutar el escenario de mock `REJECTED` de extremo a extremo (confirmación de pago → error
      síncrono → estado Fallido) y confirmar que HU01-S6 recibe los datos correctos. Verificar con la
      prueba de integración/e2e de este escenario.
- [ ] 4.2 Ejecutar el escenario de mock de saldo insuficiente (`409 INSUFFICIENT_FUNDS`) de extremo a
      extremo y confirmar el mensaje distintivo definido en la tarea 1.4. Verificar con la prueba de
      integración/e2e de este escenario.
- [ ] 4.3 Ejecutar el escenario de vencimiento del tiempo máximo de "Procesando" de extremo a extremo
      (simulando un backend que nunca resuelve un estado terminal) y confirmar que se alcanza el estado
      Expirado definido en la tarea 2.2. Verificar con la prueba de integración/e2e de este escenario.
- [ ] 4.4 Confirmar que la cobertura del código modificado en `web/` para esta slice alcanza ≥ 85%.
      Verificar con el reporte de cobertura de `python tools/lab.py test web` ejecutado desde la raíz.
- [ ] 4.5 Registrar evidencia en el documento general de hallazgos (sección `web`, HU01-S3) con el
      resultado de las pruebas y la cobertura obtenida, conservando las secciones existentes del
      documento.

## Workflow follow-up

- Antes de iniciar la tarea 1.1, confirmar que HU01-S1 (camino feliz y *polling* base) ya está
  implementado y disponible (como código, o al menos como change de OpenSpec aprobado); si no lo está,
  resolverlo primero.
- Antes de cerrar la tarea 2.1, validar con producto el tiempo máximo exacto de "Procesando" (ver
  `design.md` — Open Questions); hasta entonces, usar el valor placeholder sin bloquear el resto de la
  implementación.
- Archivar este change una vez aprobada la implementación, con revisión humana y pruebas reales
  ejecutadas (no inventadas), según el contrato general de agentes del repositorio. Se recomienda archivar
  en el orden HU01-S1 → HU01-S3 → HU01-S4 para que `payment-orchestration` quede consolidada
  correctamente en `openspec/specs/`.
