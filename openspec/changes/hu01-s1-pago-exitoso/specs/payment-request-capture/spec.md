# Spec Delta

## Purpose

Permite a un pagador autenticado seleccionar un contacto de una lista fija, capturar los datos de un pago
de bajo monto y confirmar un resumen antes de enviarlo a orquestación.

## ADDED Requirements

### Requirement: Selección de contacto desde lista fija
El sistema SHALL presentar al pagador una lista fija de contactos de mock (`contact-demo`,
`merchant-demo`), mostrando nombre y alias enmascarado de cada uno, sin funcionalidad de búsqueda.

#### Scenario: Pagador ve la lista fija de contactos
- **WHEN** el pagador llega a la pantalla de selección de contacto desde Home
- **THEN** ve la lista fija de contactos de mock (`contact-demo`, `merchant-demo`) con nombre y alias
  enmascarado, sin ningún campo de búsqueda

### Requirement: Captura de monto del pago
El sistema SHALL capturar el monto del pago en soles mediante teclado numérico, usando `wallet-demo` como
origen de solo lectura y el contacto seleccionado como destino, y SHALL convertir el monto a céntimos antes
de enviarlo al backend.

#### Scenario: Monto en soles se convierte a céntimos
- **WHEN** el pagador ingresa un monto en soles en la pantalla de datos del pago y continúa
- **THEN** el monto se convierte a céntimos antes de ser incluido en la solicitud enviada al backend

### Requirement: Confirmación previa con resumen y acciones
El sistema SHALL mostrar, antes de enviar el pago, un resumen en texto simple del monto y el destinatario
junto con las acciones **Confirmar** y **Corregir**.

#### Scenario: Resumen de confirmación previa a enviar el pago
- **WHEN** el pagador seleccionó un contacto de la lista fija y capturó un monto válido
- **THEN** llega a una pantalla de confirmación previa que muestra un resumen en texto simple (por ejemplo,
  "Vas a pagar S/ 15.00 a Juan Pérez") con los botones **Confirmar** y **Corregir**
