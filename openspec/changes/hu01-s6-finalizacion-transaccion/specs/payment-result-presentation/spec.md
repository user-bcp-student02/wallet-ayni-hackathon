# Spec Delta

## Purpose

Presentar de forma consistente el estado intermedio "Procesando" y los tres estados terminales de un pago
(Exitoso, Fallido, Expirado) a partir de datos ya resueltos por quien invoca el componente, sin decidir por
sí mismo qué estado corresponde.

## ADDED Requirements

### Requirement: Presentación del estado Exitoso
El sistema SHALL, al recibir un estado Exitoso junto con los datos del pago (`id`, fecha, destino, monto),
renderizar esos cuatro campos junto con el mensaje de éxito correspondiente del catálogo de mensajes.

#### Scenario: Renderizado con datos de estado Exitoso
- **WHEN** el componente recibe el estado Exitoso y los datos del pago (`id`, fecha, destino, monto)
- **THEN** se muestran los cuatro campos junto con el mensaje de éxito del catálogo de mensajes

### Requirement: Presentación de los estados Fallido y Expirado
El sistema SHALL, al recibir un estado Fallido o Expirado junto con los datos del pago (`id`, fecha,
destino, monto), renderizar los mismos cuatro campos que en el estado Exitoso, junto con el mensaje
correspondiente a ese estado.

#### Scenario: Renderizado con datos de estado Fallido
- **WHEN** el componente recibe el estado Fallido y los datos del pago (`id`, fecha, destino, monto)
- **THEN** se muestran los cuatro campos junto con el mensaje de Fallido del catálogo de mensajes

#### Scenario: Renderizado con datos de estado Expirado
- **WHEN** el componente recibe el estado Expirado y los datos del pago (`id`, fecha, destino, monto)
- **THEN** se muestran los cuatro campos junto con el mensaje de Expirado del catálogo de mensajes

### Requirement: Estado intermedio Procesando resiliente
Mientras el resultado del pago no esté resuelto en un estado terminal, el sistema SHALL mostrar el mensaje
tranquilizador del estado "Procesando" en lugar de los datos finales del pago, y SHALL conservar ese
intento visible si el pagador cierra y reabre la app antes de que se resuelva.

#### Scenario: Estado Procesando mientras el resultado no se resuelve
- **WHEN** el componente recibe el estado "Procesando" (resultado aún no resuelto)
- **THEN** se muestra el mensaje tranquilizador del catálogo de mensajes en lugar de los datos finales del
  pago

#### Scenario: El intento no se pierde al cerrar y reabrir la app
- **WHEN** el pagador cierra y reabre la app mientras el pago sigue en estado "Procesando"
- **THEN** la app vuelve a mostrar el estado "Procesando" para ese mismo intento, en lugar de perderlo

### Requirement: Navegación "Volver al inicio" desde cualquier estado terminal
El sistema SHALL ofrecer la acción **Volver al inicio** en cualquier estado terminal mostrado (Exitoso,
Fallido o Expirado), y SHALL navegar a Home al activarla.

#### Scenario: Volver al inicio navega a Home
- **WHEN** el pagador presiona **Volver al inicio** estando en cualquier estado terminal (Exitoso, Fallido
  o Expirado)
- **THEN** la app navega a Home

### Requirement: El componente no decide el estado ni llama al backend
El sistema SHALL recibir el estado terminal (o "Procesando") y los datos del pago ya resueltos por quien
invoca el componente, y NO SHALL realizar llamadas propias a `POST /payments` ni a `GET /payments/{id}`, ni
mapear códigos de error HTTP para decidir el estado a mostrar.

#### Scenario: El componente no realiza llamadas propias al backend de pagos
- **WHEN** el componente es invocado con un estado y datos ya resueltos
- **THEN** no emite ninguna solicitud propia a `POST /payments` ni a `GET /payments/{id}`, ni interpreta
  códigos de error HTTP para determinar el estado
