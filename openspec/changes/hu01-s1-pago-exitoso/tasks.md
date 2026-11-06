# Tasks

Alcance: pantallas y orquestación de `web/` para HU01-S1 (capabilities `payment-request-capture` y
`payment-orchestration`). El contrato de `POST /payments` / `GET /payments/{id}` (real o mock) es un
prerrequisito externo: ver `## Workflow follow-up`. Estas tareas no deciden ni implementan ese backend.

## 1. Selección de contacto (T2) — `payment-request-capture`

- [ ] 1.1 Implementar la pantalla de selección de contacto que lista `contact-demo` y `merchant-demo` con
      nombre y alias enmascarado, reutilizando los componentes base y el catálogo de mensajes de
      HU01-S0, sin ningún campo de búsqueda. Verificar con una prueba de componente que renderiza ambos
      contactos y confirma la ausencia de un buscador.
- [ ] 1.2 Conectar la navegación desde el botón "Nuevo pago" de Home (HU01-S5) hacia esta pantalla.
      Verificar con una prueba de routing/navegación.

## 2. Datos del pago (T3) — `payment-request-capture`

- [ ] 2.1 Implementar la pantalla de datos del pago con origen de solo lectura `wallet-demo`, destino el
      contacto elegido en el paso 1, y captura del monto en soles con teclado numérico. Verificar con una
      prueba de componente.
- [ ] 2.2 Implementar la conversión del monto capturado en soles a céntimos antes de pasarlo al paso de
      confirmación. Verificar con una prueba unitaria que cubra al menos un caso con decimales (por
      ejemplo, S/ 15.50 → 1550 céntimos).

## 3. Confirmación previa (T4/CA1) — `payment-request-capture`

- [ ] 3.1 Implementar la pantalla de confirmación previa con el resumen en texto simple (monto y
      destinatario interpolados, por ejemplo "Vas a pagar S/ 15.00 a Juan Pérez") y los botones
      **Confirmar**/**Corregir**, usando el catálogo de mensajes de HU01-S0. Verificar con una prueba de
      componente que confirma la interpolación correcta del resumen.
- [ ] 3.2 Implementar la acción **Corregir**, que regresa al pagador a la pantalla de datos del pago (T3)
      conservando el contacto ya seleccionado. Verificar con una prueba de navegación.

## 4. Orquestación del pago (T7, camino feliz) — `payment-orchestration`

- [ ] 4.1 Implementar la generación de un `Idempotency-Key` único por wallet (1–200 caracteres) y su
      envío como header en `POST /payments`. Verificar con una prueba unitaria que confirma el rango de
      longitud y que dos pagos del mismo wallet generan claves distintas.
- [ ] 4.2 Implementar el bloqueo del botón **Confirmar** desde el envío de `POST /payments` hasta obtener
      un resultado terminal (protección de doble toque). Verificar con una prueba que confirme que un
      segundo click durante el procesamiento no dispara una segunda solicitud.
- [ ] 4.3 Implementar el *polling* de `GET /payments/{id}` tras recibir `202 PENDING`, hasta alcanzar el
      estado terminal `APPROVED`. Verificar con una prueba que, dado un mock que resuelve `APPROVED` tras
      N consultas, confirme que el polling se detiene y usa esos datos.
- [ ] 4.4 Implementar el mapeo de campos entre el cuerpo camelCase de la solicitud (`walletId`,
      `amountCents`) y la respuesta snake_case del backend (`wallet_id`, `amount`, etc.). Verificar con
      una prueba unitaria del mapeo en ambas direcciones.
- [ ] 4.5 Invocar el componente de resultado de HU01-S6 con el estado "Procesando" mientras el pago no se
      resuelve, y con el estado Exitoso más los datos del pago (`id`, fecha, destino, monto) al alcanzar
      `APPROVED`. Verificar con una prueba que confirme ambas invocaciones con los argumentos correctos.

## 5. Verificación end-to-end

- [ ] 5.1 Ejecutar el escenario de mock `APPROVED` de extremo a extremo (selección de contacto → monto →
      confirmación → Confirmar → Procesando → Exitoso) contra un backend real o mock disponible, y
      confirmar que HU01-S6 recibe los datos correctos. Verificar con la prueba de integración/e2e de
      este escenario.
- [ ] 5.2 Confirmar que la cobertura del código modificado en `web/` para estas pantallas y la
      orquestación alcanza ≥ 85%. Verificar con el reporte de cobertura de `python tools/lab.py test web`
      ejecutado desde la raíz.
- [ ] 5.3 Registrar evidencia en el documento general de hallazgos (sección `web`, HU01-S1) con el
      resultado de las pruebas y la cobertura obtenida, conservando las secciones existentes del
      documento.

## Workflow follow-up

- Antes de iniciar la tarea 1.1 (o en paralelo, si ya está resuelto), confirmar que `POST /payments` y
  `GET /payments/{id}` están disponibles — reales o mock — con el contrato descrito en el Alcance de
  `HU01-S1-pago-exitoso.md`; si no lo están, resolverlo en un change separado antes de continuar (ver
  `design.md` — Open Questions).
- Archivar este change una vez aprobada la implementación, con revisión humana y pruebas reales
  ejecutadas (no inventadas), según el contrato general de agentes del repositorio.
