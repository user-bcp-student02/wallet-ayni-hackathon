# HU01-S4 — Caso especial 504: timeout tras procesamiento ya aprobado

## Historia de usuario

Como pagador,
quiero que el sistema no me muestre un resultado de error cuando el backend tarda en responder pero mi
pago ya fue aprobado por el proveedor,
para no creer que perdí mi dinero cuando en realidad el pago sí se realizó.

## Origen y trazabilidad

- Slice derivada de **HU01 — Confirmación de pagos de bajo monto** ([HU01-payment.md](./HU01-payment.md)).
- Cubre exclusivamente el caso especial **`504 TIMEOUT_AFTER_PROCESSING`** descrito en **T7 (Orquestación
  del pago)**, y la nota técnica de mapeo de campos camelCase/snake_case necesaria para interpretarlo
  correctamente.
- Técnica de slicing aplicada: aislar la regla más riesgosa y asimétrica de la HU original (un `504` que
  **no** debe tratarse como fallo, a diferencia de todos los demás códigos de error cubiertos en
  **HU01-S3**), para poder implementarla, revisarla y probarla sin mezclar su lógica con el resto del
  manejo de errores.

## Alcance

- Ante una respuesta `504 TIMEOUT_AFTER_PROCESSING` de `POST /payments` o de `GET /payments/{id}` durante
  el *polling*, la app:
  - **No** muestra "Expirado" ni "Fallido" directamente por ese `504`.
  - Continúa el *polling* de `GET /payments/{id}` hasta alcanzar un estado terminal real (`APPROVED` o
    `REJECTED`), respetando el tiempo máximo de la app ya definido en **HU01-S3** para esa continuación.
- Normalización del mapeo de campos entre el cuerpo de la petición (camelCase: `walletId`, `amountCents`)
  y las respuestas del backend (snake_case: `wallet_id`, `amount`, etc.), en la medida necesaria para que
  el *polling* posterior al `504` interprete correctamente el estado recibido.

## Criterios de aceptación

1. **Dado** que `POST /payments` o `GET /payments/{id}` responden `504 TIMEOUT_AFTER_PROCESSING`,
   **cuando** la app recibe esa respuesta, **entonces** no muestra "Expirado" ni "Fallido" de inmediato.
2. **Dado** el escenario anterior, **cuando** la app continúa el *polling* de `GET /payments/{id}`,
   **entonces** eventualmente refleja el estado terminal real del pago (Exitoso o Fallido, según
   corresponda), sin mostrar un resultado incorrecto.
3. **Dado** que el *polling* posterior a un `504` agota el tiempo máximo de la app sin resolver, **cuando**
   se cumple ese plazo, **entonces** se aplica la misma regla de "Expirado" definida en **HU01-S3** (el
   `504` no introduce una regla de expiración distinta).
4. **Dado** una respuesta de `GET /payments/{id}` en snake_case tras un `504`, **cuando** la app la procesa,
   **entonces** la interpreta correctamente gracias al mapeo de campos, sin confundir el estado por un
   problema de formato.

## Dependencias

- Depende de **HU01-S1** (reutiliza el *polling* base de T7).
- Depende de **HU01-S3** (reutiliza el esqueleto de manejo de errores/resultados terminales y la regla de
  tiempo máximo para "Expirado").
- No tiene dependientes: es la última slice del backlog derivado de HU01.

## Endpoints referenciados

| Endpoint | Uso |
|---|---|
| `POST /payments` | Puede responder `504 TIMEOUT_AFTER_PROCESSING` aunque el pago ya se haya creado |
| `GET /payments/{id}` | *Polling* que debe continuar tras un `504` hasta el estado terminal real |

> No existe en el contrato un endpoint de resolución forzada para producción; la única forma de conocer el
> estado terminal real tras un `504` es seguir el *polling* de `GET /payments/{id}`.

## Fuera de alcance de esta porción

- Cualquier otro código de error (`409/400/404/413/429/503`): cubiertos en **HU01-S3**, que trata errores
  simétricos (si ocurren, el pago sí falló).
- Endpoints de control de mock (`GET /control/events`, `POST /callbacks/payments`,
  `POST /control/provider/{id}/complete`): son herramientas para preparar escenarios de prueba, no parte
  del flujo de producción de esta app.
- Reintentos y reutilización de `Idempotency-Key`: backlog de la HU original, sin cambios.

## Supuestos heredados de HU01

- El contrato de backend (mock) no incluye un endpoint de resolución forzada de pagos
  (`POST /payments/{id}/resolve`); la única forma de conocer el estado terminal en producción es el
  *polling* de `GET /payments/{id}`.

## Pruebas asociadas (subconjunto de T12 original)

- Escenario de mock `DELAYED_APPROVAL` (polling prolongado antes de resolver).
- Escenario de mock `TIMEOUT_AFTER_PROCESSING`: verificar que, tras el `504`, el polling continúa y
  finalmente refleja el estado real (no un falso "Expirado"/"Fallido").
- Escenario de mock `PROVIDER_ERROR` usado como contraste negativo (debe producir Fallido, no debe
  tratarse con esta misma lógica de continuación; confirma la separación de responsabilidad con HU01-S3).
