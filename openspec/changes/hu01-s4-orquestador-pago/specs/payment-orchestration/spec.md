# Spec Delta

## ADDED Requirements

### Requirement: Continuidad del polling tras 504 TIMEOUT_AFTER_PROCESSING
El sistema SHALL, al recibir `504 TIMEOUT_AFTER_PROCESSING` en la respuesta de `POST /payments` o en una
respuesta de `GET /payments/{id}` durante el *polling*, continuar el *polling* de `GET /payments/{id}` sin
mostrar de inmediato un resultado de Fallido o Expirado, hasta alcanzar un estado terminal real
(`APPROVED` o `REJECTED`).

#### Scenario: 504 en POST /payments no detiene el polling
- **WHEN** `POST /payments` responde `504 TIMEOUT_AFTER_PROCESSING`
- **THEN** la app no muestra "Expirado" ni "Fallido" de inmediato y continúa el *polling* de
  `GET /payments/{id}`

#### Scenario: 504 durante el polling no detiene el polling
- **WHEN** una consulta de `GET /payments/{id}` durante el *polling* responde
  `504 TIMEOUT_AFTER_PROCESSING`
- **THEN** la app no muestra "Expirado" ni "Fallido" de inmediato y continúa consultando
  `GET /payments/{id}`

#### Scenario: Polling continuado tras un 504 alcanza el estado terminal real
- **WHEN** el *polling* continuado tras un `504` alcanza un estado terminal (`APPROVED` o `REJECTED`)
- **THEN** se invoca el componente de presentación de resultado con ese estado terminal real y los datos
  del pago, sin haber mostrado antes un resultado incorrecto

### Requirement: Reutilización de la regla de expiración existente tras un 504
El sistema SHALL aplicar, cuando el *polling* continuado tras un `504` agota el mismo tiempo máximo de la
app definido para cualquier agotamiento de *polling*, la misma regla de "Expirado" ya existente, sin
introducir un tiempo máximo ni una regla de expiración distintos para este caso.

#### Scenario: Agotamiento del tiempo máximo tras un 504 produce Expirado
- **WHEN** el *polling* continuado después de un `504` agota el tiempo máximo de la app sin resolver un
  estado terminal
- **THEN** se invoca el componente de presentación de resultado con el estado Expirado, aplicando la misma
  regla de expiración ya definida para cualquier agotamiento de *polling*

### Requirement: Mapeo de campos correcto en respuestas posteriores a un 504
El sistema SHALL interpretar correctamente, usando el mapeo de campos camelCase/snake_case ya existente,
las respuestas de `GET /payments/{id}` recibidas durante el *polling* posterior a un `504`, sin confundir
el estado por un problema de formato.

#### Scenario: Respuesta snake_case tras un 504 se interpreta correctamente
- **WHEN** `GET /payments/{id}` devuelve una respuesta en snake_case (`wallet_id`, `amount`, etc.) durante
  el *polling* posterior a un `504`
- **THEN** la app interpreta correctamente el estado y los datos del pago gracias al mapeo de campos
  existente
