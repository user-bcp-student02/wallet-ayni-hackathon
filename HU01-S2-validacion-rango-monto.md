# HU01-S2 — Validación de rango de bajo monto

## Historia de usuario

Como pagador,
quiero que el sistema valide que el monto que ingreso esté dentro del rango permitido de "bajo monto"
antes de dejarme confirmar el pago,
para evitar enviar pagos que de todas formas serán rechazados.

## Origen y trazabilidad

- Slice derivada de **HU01 — Confirmación de pagos de bajo monto** ([HU01-payment.md](./HU01-payment.md)).
- Cubre íntegramente **CA2 — Rango de bajo monto** y **T8 — Regla de monto bajo y límites**.
- Técnica de slicing aplicada: variación de regla de negocio sobre una pantalla (T3/T4) ya construida en
  **HU01-S1**, separando la regla de validación del camino feliz para poder estimarla, implementarla y
  probarla de forma aislada.

## Alcance

- Validar que el monto ingresado en **T3 (Datos del pago)** esté entre S/ 10 (mínimo de negocio) y S/ 1000
  (máximo), tanto en la app (antes de habilitar continuar) como al orquestar el pago en el servidor
  (`POST /payments`).
- El campo de captura numérica no acepta valores menores a S/ 0.01 (piso técnico de entrada, independiente
  del mínimo de negocio de S/ 10).
- Mensaje de bloqueo claro cuando el monto está fuera de rango, mostrado antes de llegar a la pantalla de
  confirmación previa (T4), usando el catálogo de mensajes de **HU01-S0**.

## Criterios de aceptación

1. **Dado** que el pagador ingresa un monto entre S/ 10 y S/ 1000 (inclusive), **cuando** intenta
   continuar desde T3, **entonces** el sistema permite avanzar a la confirmación previa (T4) sin bloqueo.
2. **Dado** que el pagador ingresa un monto menor a S/ 10 o mayor a S/ 1000, **cuando** intenta continuar,
   **entonces** el sistema bloquea el envío antes de confirmar, mostrando un mensaje claro del motivo.
3. **Dado** que el campo de captura recibe un valor menor a S/ 0.01, **cuando** el pagador intenta
   ingresarlo, **entonces** el campo lo rechaza a nivel de entrada (piso técnico), independientemente de la
   validación de negocio del punto 2.
4. **Dado** que un monto pasó la validación de la app pero el servidor lo rechaza al orquestar el pago
   (defensa en profundidad), **cuando** ocurre esa discrepancia, **entonces** el pagador recibe el mismo
   mensaje de bloqueo claro, sin llegar a un estado "Procesando" falso.

## Dependencias

- Depende de **HU01-S1** (reutiliza las pantallas T3/T4 y la llamada a `POST /payments` ya construidas).
- Depende de **HU01-S0** (catálogo de mensajes para el texto de bloqueo).
- No es prerrequisito de HU01-S3 ni HU01-S4 (son independientes entre sí; pueden desarrollarse en
  paralelo una vez completada HU01-S1).

## Endpoints referenciados

| Endpoint | Uso |
|---|---|
| `POST /payments` | Punto donde el servidor valida nuevamente el rango de monto (defensa en profundidad) |

## Fuera de alcance de esta porción

- Configuración de límites de bajo monto por proveedor/canal: sigue como **pregunta abierta** de la HU
  original, no se cierra en esta slice.
- Cualquier otra regla de negocio o manejo de error de `POST /payments` ajeno al rango de monto (ver
  **HU01-S3** y **HU01-S4**).

## Supuestos heredados de HU01

- Monto mínimo de negocio para "bajo monto": S/ 10. Monto máximo: S/ 1000. Piso técnico de captura:
  S/ 0.01.

## Preguntas abiertas heredadas de HU01

- Configuración final de límites de bajo monto por proveedor/canal (T8).

## Pruebas asociadas (subconjunto de T12 original)

- Montos en el borde inferior (S/ 10, S/ 9.99, S/ 0.01) y superior (S/ 1000, S/ 1000.01) del rango.
- Monto rechazado por el servidor aunque haya pasado la validación de la app (caso de discrepancia).
