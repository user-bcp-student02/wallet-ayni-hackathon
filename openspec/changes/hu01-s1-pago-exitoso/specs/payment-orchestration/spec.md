# Spec Delta

## Purpose

Toma una solicitud de pago confirmada y la lleva a un resultado terminal, integrando con el backend de
pagos y comunicando el estado resuelto (o intermedio) a la pantalla de finalización.

## ADDED Requirements

### Requirement: Bloqueo de doble toque durante el procesamiento
El sistema SHALL bloquear el botón de confirmar un pago desde el momento en que el pagador lo presiona
hasta obtener un resultado terminal, para evitar envíos duplicados por doble toque.

#### Scenario: Botón de confirmar queda bloqueado mientras se procesa
- **WHEN** el pagador presiona **Confirmar** en la pantalla de confirmación previa
- **THEN** el botón de confirmar queda bloqueado hasta que el pago alcanza un resultado terminal

### Requirement: Idempotency-Key obligatoria en el envío del pago
El sistema SHALL generar un `Idempotency-Key` de entre 1 y 200 caracteres, único por wallet, y SHALL
enviarlo como header obligatorio en cada `POST /payments`.

#### Scenario: Envío de POST /payments con Idempotency-Key
- **WHEN** el pagador confirma el pago y se envía `POST /payments`
- **THEN** la solicitud incluye un header `Idempotency-Key` de 1 a 200 caracteres, único por wallet

### Requirement: Polling hasta estado terminal aprobado
El sistema SHALL, tras recibir `202 PENDING` de `POST /payments`, consultar `GET /payments/{id}` en forma
repetida hasta alcanzar el estado terminal `APPROVED`, y al alcanzarlo SHALL invocar el componente de
presentación de resultado con el estado Exitoso y los datos del pago (`id`, fecha, destino, monto).

#### Scenario: Polling detecta APPROVED e invoca el resultado Exitoso
- **WHEN** `POST /payments` responde `202 PENDING` y un `GET /payments/{id}` posterior devuelve el estado
  terminal `APPROVED`
- **THEN** se invoca el componente de presentación de resultado con el estado Exitoso y los datos del pago
  (`id`, fecha, destino, monto)

### Requirement: Estado intermedio Procesando mientras no hay resultado terminal
Mientras el backend no haya resuelto el pago en un estado terminal, el sistema SHALL invocar el componente
de presentación de resultado con el estado intermedio "Procesando".

#### Scenario: Pago en curso muestra el estado Procesando
- **WHEN** el pago está en curso y el backend aún no resuelve un estado terminal
- **THEN** se invoca el componente de presentación de resultado con el estado "Procesando"

### Requirement: Normalización de campos entre request y response
El sistema SHALL enviar el cuerpo de `POST /payments` en camelCase (`walletId`, `amountCents`) y SHALL
interpretar las respuestas del backend en snake_case (`wallet_id`, `amount`, etc.), normalizando el mapeo
entre ambas convenciones.

#### Scenario: Cuerpo de la solicitud y respuesta usan convenciones distintas
- **WHEN** se arma el cuerpo de `POST /payments` y se interpreta la respuesta de `GET /payments/{id}`
- **THEN** el cuerpo de la solicitud usa camelCase (`walletId`, `amountCents`) y los campos de la
  respuesta se leen en snake_case (`wallet_id`, `amount`, etc.) sin error de mapeo
