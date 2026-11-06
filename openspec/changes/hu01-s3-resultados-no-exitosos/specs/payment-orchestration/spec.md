# Spec Delta

## ADDED Requirements

### Requirement: Mapeo de errores síncronos de POST /payments a resultado Fallido
El sistema SHALL, cuando `POST /payments` responde con `409 INSUFFICIENT_FUNDS`, `409 WALLET_NOT_ACTIVE`,
`400`, `404`, `413`, `429` o `503 PROVIDER_ERROR` antes o durante el envío del pago, invocar el componente
de presentación de resultado con el estado Fallido y los datos disponibles del pago (`id` si existe,
fecha, destino, monto).

#### Scenario: Error síncrono de POST /payments produce Fallido
- **WHEN** `POST /payments` responde con `409 INSUFFICIENT_FUNDS`, `409 WALLET_NOT_ACTIVE`, `400`, `404`,
  `413`, `429` o `503 PROVIDER_ERROR`
- **THEN** se invoca el componente de presentación de resultado con el estado Fallido y los datos
  disponibles del pago (`id` si existe, fecha, destino, monto)

#### Scenario: 503 PROVIDER_ERROR se trata como Fallido, no como Expirado
- **WHEN** `POST /payments` responde `503 PROVIDER_ERROR`, confirmando que el backend no creó ni
  operación ni reserva
- **THEN** se invoca el componente de presentación de resultado con el estado Fallido, sin iniciar ni
  continuar el *polling* de `GET /payments/{id}`

### Requirement: Resultado Expirado por agotamiento del polling
El sistema SHALL, cuando el *polling* de `GET /payments/{id}` no alcanza un estado terminal dentro de un
tiempo máximo propio de la app, invocar el componente de presentación de resultado con el estado Expirado
y los mismos datos disponibles del pago (`id` si existe, fecha, destino, monto).

#### Scenario: Agotamiento del tiempo máximo de polling produce Expirado
- **WHEN** el *polling* de `GET /payments/{id}` no resuelve un estado terminal dentro del tiempo máximo
  definido por la app
- **THEN** se invoca el componente de presentación de resultado con el estado Expirado y los datos
  disponibles del pago (`id` si existe, fecha, destino, monto)

### Requirement: Acción única tras un resultado no exitoso
El sistema SHALL ofrecer únicamente la acción "Volver al inicio" tras entregar un estado Fallido o
Expirado al componente de presentación de resultado, y SHALL NOT ofrecer una acción de "Reintentar".

#### Scenario: Resultado Fallido o Expirado ofrece solo Volver al inicio
- **WHEN** el componente de presentación de resultado recibe el estado Fallido o Expirado
- **THEN** la única acción ofrecida es "Volver al inicio"; no se ofrece la acción "Reintentar"

### Requirement: Mensaje distintivo para saldo insuficiente
El sistema SHALL, al mapear un error `409 INSUFFICIENT_FUNDS` al estado Fallido, usar un mensaje del
catálogo de mensajes que distinga este caso de un error genérico.

#### Scenario: 409 INSUFFICIENT_FUNDS muestra un mensaje distinto al genérico
- **WHEN** `POST /payments` responde `409 INSUFFICIENT_FUNDS` y se mapea al estado Fallido
- **THEN** el mensaje mostrado, obtenido del catálogo de mensajes, distingue este caso de un mensaje de
  error genérico
