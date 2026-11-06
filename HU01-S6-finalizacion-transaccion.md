# HU01-S6 — Pantalla de finalización de transacción de pago

## Historia de usuario

Como pagador,
quiero ver una pantalla de finalización clara y consistente sin importar si mi pago terminó exitoso,
fallido o expiró,
para entender el resultado de mi transacción y poder continuar.

## Origen y trazabilidad

- Slice derivada de **HU01 — Confirmación de pagos de bajo monto** ([HU01-payment.md](./HU01-payment.md)).
- Cubre íntegramente **T5 (Pantalla de resultado)** como componente de presentación único, antes repartido
  entre **HU01-S1** (solo caso Exitoso) y **HU01-S3** (casos Fallido/Expirado).
- Técnica de slicing aplicada: variación de interfaz — se separa la **pantalla** (qué se muestra y cómo)
  de la **lógica de negocio** que decide qué resultado corresponde (que permanece en HU01-S1 para el caso
  Exitoso, en HU01-S3 para el mapeo de errores/expiración, y en HU01-S4 para el caso especial `504`).
- **Impacto en el backlog existente:** HU01-S1 y HU01-S3 se actualizaron para reutilizar este componente en
  lugar de describir la pantalla de resultado cada una por su cuenta (ver notas en esos archivos).

## Alcance

- Estado intermedio **"Procesando"**, mostrado mientras el resultado aún no se resuelve, con mensaje
  tranquilizador (CA3, T11) y sin perder el intento si se cierra la app.
- Los tres estados terminales posibles — **Exitoso**, **Fallido**, **Expirado** — renderizados por el mismo
  componente, mostrando en todos los casos: referencia (el `id` del pago devuelto por el backend), fecha,
  destino y monto.
- Acción **Volver al inicio**, disponible en la pantalla de resultado, que navega a Home (**HU01-S5**).
- Textos de cada estado obtenidos del catálogo de mensajes de **HU01-S0** (sin contenido final de copy).
- La pantalla es "tonta" respecto al negocio: recibe el estado terminal (o "Procesando") y los datos del
  pago ya resueltos por quien la invoca (HU01-S1, HU01-S3 o HU01-S4); no decide por sí misma qué estado
  corresponde ni interpreta códigos de error HTTP.

## Criterios de aceptación

1. **Dado** un resultado con estado Exitoso y sus datos (`id`, fecha, destino, monto), **cuando** se
   renderiza la pantalla de finalización, **entonces** se muestran esos datos junto con el mensaje de
   éxito correspondiente del catálogo.
2. **Dado** un resultado con estado Fallido o Expirado y sus datos, **cuando** se renderiza, **entonces**
   se muestran los mismos campos (`id`, fecha, destino, monto) junto con el mensaje correspondiente.
3. **Dado** el estado intermedio "Procesando", **cuando** el resultado aún no está resuelto, **entonces**
   se muestra el mensaje tranquilizador en lugar de los datos finales, y el intento no se pierde si el
   pagador cierra y reabre la app.
4. **Dado** cualquier estado terminal mostrado, **cuando** el pagador presiona **Volver al inicio**,
   **entonces** navega a Home (HU01-S5).
5. **Dado** que esta pantalla recibe un estado y datos ya resueltos, **cuando** se le invoca, **entonces**
   no realiza llamadas propias a `POST /payments` ni a `GET /payments/{id}` ni mapea códigos de error: esa
   responsabilidad es de quien la invoca.

## Dependencias

- Depende de **HU01-S0** (componentes accesibles y catálogo de mensajes).
- Depende de **HU01-S5** (destino de la navegación "Volver al inicio").
- Es un componente proveído para **HU01-S1** (estado Exitoso), **HU01-S3** (estados Fallido/Expirado y
  mapeo de errores) y **HU01-S4** (caso especial `504`): esas slices deciden el estado y los datos; esta
  HU solo los presenta.

## Fuera de alcance de esta porción

- Determinar qué estado corresponde a cada respuesta del backend (mapeo de `409/400/404/413/429/503`,
  regla de expiración por tiempo máximo, caso especial `504`): cubierto por **HU01-S1**, **HU01-S3** y
  **HU01-S4** respectivamente.
- *Polling* de `GET /payments/{id}` y envío de `POST /payments`: fuera de esta HU, que solo presenta el
  resultado ya resuelto.
- Acción **Reintentar**: backlog de la HU original, sin cambios.
- Comprobante (`GET /payments/{id}/receipt`) e historial: backlog de la HU original, sin cambios.

## Supuestos

- El contrato de backend solo expone un `receiptId` propio a través de `GET /payments/{id}/receipt`
  (endpoint en backlog); por eso la referencia visible en esta pantalla es el `id` del pago, no un
  `receiptId` (heredado de CA3 de la HU original).

## Preguntas abiertas (nuevas, no heredadas de HU01)

- La HU original describe la acción **Volver al inicio** explícitamente para los casos Fallido/Expirado
  (CA4), mientras que la sección de flujo de T5 la lista a nivel de pantalla sin distinguir por estado. Esta
  HU **asume** que la acción está disponible también en el caso Exitoso, por consistencia de UI; queda
  pendiente confirmar esta asunción con producto/diseño.

## Preguntas abiertas heredadas de HU01

- Etiquetas finales de los estados terminales y contenido definitivo del catálogo de mensajes (T11).
- Tiempo máximo exacto que debe transcurrir en "Procesando" antes de declarar "Expirado" (decidido por
  quien invoca esta pantalla, no por ella).

## Pruebas asociadas

- Renderizado del componente con datos de estado Exitoso.
- Renderizado del componente con datos de estado Fallido.
- Renderizado del componente con datos de estado Expirado.
- Renderizado del estado intermedio "Procesando" y persistencia del intento al cerrar/reabrir la app.
- Navegación de "Volver al inicio" hacia Home.
