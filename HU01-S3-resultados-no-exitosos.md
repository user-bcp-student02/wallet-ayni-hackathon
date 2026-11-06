# HU01-S3 — Resultados no exitosos y mapeo de errores síncronos

## Historia de usuario

Como pagador,
quiero recibir un resultado claro (Fallido o Expirado) con una acción para continuar cuando mi pago no se
complete con éxito,
para saber qué pasó con mi dinero y no quedar bloqueado en la app.

## Origen y trazabilidad

- Slice derivada de **HU01 — Confirmación de pagos de bajo monto** ([HU01-payment.md](./HU01-payment.md)).
- Cubre **CA3 — Resultado único terminal** (ramas Fallido/Expirado), **CA4 — Acción tras resultado no
  exitoso**, y la parte de **T7 (Orquestación)** referida al mapeo de errores síncronos y al timeout propio
  de la app.
- Técnica de slicing aplicada: separar la rama "feliz" (**HU01-S1**) de las variantes de error, que
  concentran la mayor complejidad del criterio original CA3/CA4/T7 al agrupar tres resultados terminales y
  múltiples códigos de error en un solo criterio.
- **Nota de actualización:** la presentación de la pantalla de resultado (T5) se extrajo a
  **HU01-S6** (pantalla de finalización de transacción); esta HU solo decide el estado (Fallido/Expirado)
  y los datos, e invoca ese componente en lugar de describir la pantalla por su cuenta.

## Alcance

- **Resultado Fallido:** mapeo de los siguientes errores de `POST /payments` a un resultado Fallido claro,
  antes o durante el envío:
  - `409 INSUFFICIENT_FUNDS`
  - `409 WALLET_NOT_ACTIVE`
  - `400` (solicitud inválida)
  - `404` (recurso no encontrado)
  - `413` (payload demasiado grande)
  - `429` (límite de tasa excedido)
  - `503 PROVIDER_ERROR`: el backend confirma que no se creó operación ni reserva, por lo que corresponde
    un resultado terminal de tipo Fallido (a diferencia del `504`, cubierto en **HU01-S4**).
- **Resultado Expirado:** si el *polling* de `GET /payments/{id}` no alcanza un estado terminal dentro de
  un tiempo máximo propio de la app, se declara "Expirado" (el tiempo máximo exacto queda como pregunta
  abierta, ver más abajo).
- Al determinar Fallido o Expirado, invocar el componente de **HU01-S6** con ese estado y los datos del
  pago (`id` si existe, fecha, destino, monto); esta HU no describe la pantalla en sí (ver **HU01-S6**).
- Toda la lógica se apoya en los componentes y el catálogo de mensajes de **HU01-S0**.

## Criterios de aceptación

1. **Dado** que `POST /payments` responde con alguno de los errores síncronos listados (`409`, `400`,
   `404`, `413`, `429`) o `503 PROVIDER_ERROR`, **cuando** ocurre antes o durante el envío, **entonces** se
   invoca el componente de **HU01-S6** con estado Fallido y los datos disponibles (`id` si existe, fecha,
   destino, monto).
2. **Dado** que el *polling* de `GET /payments/{id}` no resuelve un estado terminal dentro del tiempo
   máximo definido por la app, **cuando** se cumple ese tiempo, **entonces** se invoca el componente de
   **HU01-S6** con estado Expirado y los mismos datos.
3. **Dado** un estado Fallido o Expirado entregado a **HU01-S6**, **cuando** se muestra la pantalla de
   resultado, **entonces** se ofrece la acción **Volver al inicio** (CA4), provista por ese componente; no
   se ofrece la acción **Reintentar** (queda en backlog).
4. **Dado** un error `409 INSUFFICIENT_FUNDS`, **cuando** se mapea a Fallido, **entonces** el mensaje
   mostrado distingue este caso de un error genérico (usando el catálogo de mensajes de HU01-S0), aunque
   la etiqueta final de texto no esté cerrada con producto.

## Dependencias

- Depende de **HU01-S1** (reutiliza la base de T7 y el *polling* construidos en el camino feliz).
- Depende de **HU01-S0** (catálogo de mensajes y componentes accesibles).
- Depende de **HU01-S6** (la presentación del resultado Fallido/Expirado, incluida la acción "Volver al
  inicio", la provee ese componente; esta HU solo decide el estado y los datos).
- Es prerrequisito de **HU01-S4**: el manejo especial de `504` se apoya en el esqueleto de manejo de
  errores y estados terminales construido aquí.

## Endpoints referenciados

| Endpoint | Uso |
|---|---|
| `POST /payments` | Origen de los errores síncronos mapeados (`409/400/404/413/429/503`) |
| `GET /payments/{id}` | *Polling* cuyo vencimiento produce el resultado Expirado |

## Fuera de alcance de esta porción

- `504 TIMEOUT_AFTER_PROCESSING`: tiene un comportamiento asimétrico (no es un fallo real) y se trata por
  separado en **HU01-S4** para no mezclarlo con el resto de errores, que sí son fallos simétricos.
- Acción **Reintentar** y toda reutilización de `Idempotency-Key` entre intentos (incluye el manejo del
  error `409 IDEMPOTENCY_CONFLICT`): backlog de la HU original, sin cambios.
- Validación de rango de monto (ver **HU01-S2**): esta slice asume que el monto ya es válido y se enfoca
  solo en errores de orquestación.

## Preguntas abiertas heredadas de HU01

- Tiempo máximo exacto que debe transcurrir en "Procesando" antes de declarar "Expirado".
- Etiquetas finales de los estados terminales (Fallido/Expirado) y contenido definitivo del catálogo de
  mensajes.

## Pruebas asociadas (subconjunto de T12 original)

- Escenario de mock `REJECTED`.
- Saldo insuficiente (`409 INSUFFICIENT_FUNDS`).
- Vencimiento del tiempo máximo de "Procesando" → resultado Expirado.
