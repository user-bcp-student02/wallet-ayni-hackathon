# HU01-S1 — Pago exitoso de bajo monto (camino feliz)

## Historia de usuario

Como pagador,
quiero completar un pago de bajo monto válido a un contacto y ver la confirmación de que fue exitoso,
para confiar en que mi dinero llegó a destino.

## Origen y trazabilidad

- Slice derivada de **HU01 — Confirmación de pagos de bajo monto** ([HU01-payment.md](./HU01-payment.md)).
- Cubre el camino feliz de **T2 (selección de contacto)**, **T3 (datos del pago)**,
  **T4/CA1 (confirmación previa)** y **T7 (orquestación, solo rama exitosa)**.
- Técnica de slicing aplicada: *workflow steps* — se recorta el flujo completo a la ruta sin variantes de
  error, para obtener el primer extremo-a-extremo verificable ("walking skeleton").
- **Nota de actualización:** **T1 (Home)** y **T5 (pantalla de resultado)** se extrajeron a HU
  independientes — **HU01-S5** (Home: entrada a la funcionalidad de pagos) y **HU01-S6** (pantalla de
  finalización de transacción) — para evitar duplicar esas pantallas entre slices; esta HU ahora empieza
  en T2 y termina invocando el componente de HU01-S6 con el estado Exitoso.

## Alcance

- **T2. Selección de contacto:** lista fija de contactos de mock (`contact-demo`, `merchant-demo`) con
  nombre y alias enmascarado. No incluye búsqueda (ver **Fuera de alcance**).
- **T3. Datos del pago:** origen de solo lectura (`wallet-demo`), destino el contacto elegido, monto en
  soles capturado con teclado numérico y convertido a céntimos antes de enviarlo al backend.
- **T4. Confirmación previa (CA1):** resumen en texto simple (p. ej. "Vas a pagar S/ 15.00 a Juan Pérez")
  con botones **Confirmar** y **Corregir**.
- **T7. Orquestación — solo camino feliz:**
  - Bloquear el botón de confirmar mientras se procesa (protección básica de doble toque).
  - Generar un `Idempotency-Key` (1–200 caracteres, único por wallet) y enviarlo como header obligatorio en
    `POST /payments`.
  - Tras `POST /payments` (`202 PENDING`), hacer *polling* de `GET /payments/{id}` hasta alcanzar el estado
    terminal `APPROVED`.
  - Normalizar el mapeo de campos: el cuerpo de `POST /payments` usa camelCase (`walletId`, `amountCents`)
    y las respuestas usan snake_case (`wallet_id`, `amount`, etc.).
  - Al alcanzar `APPROVED`, invocar el componente de **HU01-S6** con el estado Exitoso y los datos del
    pago (`id`, fecha, destino, monto); mientras no se resuelve, invocarlo con el estado "Procesando".
- Todas las pantallas se construyen sobre los componentes y el catálogo de mensajes de **HU01-S0**.

## Criterios de aceptación

1. **Dado** que el pagador llega desde Home (**HU01-S5**) y selecciona un contacto de la lista fija
   (`contact-demo` o `merchant-demo`) y captura un monto, **cuando** llega a la pantalla de confirmación
   previa, **entonces** ve el resumen en texto simple con los botones **Confirmar** y **Corregir** (CA1).
2. **Dado** que el pagador presiona **Confirmar**, **cuando** se envía `POST /payments` con
   `Idempotency-Key` generado, **entonces** el botón de confirmar queda bloqueado hasta obtener un
   resultado terminal.
3. **Dado** que el backend resuelve el pago como `APPROVED`, **cuando** el polling de
   `GET /payments/{id}` detecta el estado terminal, **entonces** se invoca el componente de **HU01-S6** con
   el estado Exitoso y los datos del pago (`id`, fecha, destino, monto) (CA3 parcial).
4. **Dado** que el pago está en curso, **cuando** el backend aún no resuelve, **entonces** se invoca el
   componente de **HU01-S6** con el estado intermedio "Procesando".

## Dependencias

- Depende de **HU01-S0** (componentes y catálogo de mensajes).
- Depende de **HU01-S5** (Home es el punto de entrada a esta pantalla T2).
- Depende de **HU01-S6** (la presentación del resultado, incluido el estado "Procesando", la provee ese
  componente; esta HU solo decide el estado y los datos).
- Es prerrequisito funcional de **HU01-S2, HU01-S3 y HU01-S4**: todas reutilizan las pantallas T3/T4 y
  la orquestación T7 construidas aquí, agregando variantes de negocio y error.

## Endpoints referenciados

| Endpoint | Uso |
|---|---|
| `POST /payments` | Inicia el pago (`202 PENDING`), requiere header `Idempotency-Key` (T7) |
| `GET /payments/{id}` | *Polling* de estado hasta resultado terminal `APPROVED` (T7) |

> `GET /wallets/{id}` ya no se referencia en esta HU: pasó a **HU01-S5** (Home).

## Fuera de alcance de esta porción

- Home (T1) y la pantalla de finalización (T5): cubiertas por **HU01-S5** y **HU01-S6** respectivamente.
- Búsqueda de contactos en T2 (la lista es fija y pequeña; búsqueda no aporta valor en esta slice mínima).
- Cualquier monto fuera del rango de bajo monto, y su validación/bloqueo (ver **HU01-S2**).
- Cualquier resultado distinto de Exitoso: Fallido, Expirado, y todo mapeo de errores (ver **HU01-S3**).
- Manejo especial de `504 TIMEOUT_AFTER_PROCESSING` (ver **HU01-S4**).
- Reintentos, reutilización de `Idempotency-Key`, historial, comprobante e instrumentación (backlog de la
  HU original, sin cambios).
- Flujo de pago por QR (`qrId`): fuera de esta HU y de toda slice derivada.

## Supuestos heredados de HU01

- El origen del pago es siempre `wallet-demo` (solo lectura) y los contactos disponibles son
  `contact-demo` y `merchant-demo`, ambos con pago directo (sin `qrId`).
- El cuerpo de `POST /payments` usa camelCase y las respuestas del backend usan snake_case; el front debe
  mapear entre ambas convenciones.

## Pruebas asociadas (subconjunto de T12 original)

- Escenario de mock `APPROVED` (camino feliz completo).
- Doble toque: validar que el botón de confirmar permanece bloqueado mientras se procesa (la validación por
  `Idempotency-Key` ante reintentos queda en backlog).
