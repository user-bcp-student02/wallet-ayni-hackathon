# Spec Delta

## Purpose

Valida que el monto de un pago de bajo monto esté dentro del rango de negocio permitido (S/ 10–S/ 1000),
tanto en la app antes de confirmar como en el servidor al orquestar el pago, para evitar envíos que de
todas formas serán rechazados.

## ADDED Requirements

### Requirement: Rango de negocio permitido en la app
El sistema SHALL permitir avanzar de T3 (Datos del pago) a la confirmación previa (T4) únicamente cuando el
monto ingresado esté entre S/ 10 y S/ 1000, ambos inclusive.

#### Scenario: Monto dentro del rango permite avanzar
- **WHEN** el pagador ingresa un monto entre S/ 10 y S/ 1000 (inclusive) en T3 e intenta continuar
- **THEN** el sistema permite avanzar a la confirmación previa (T4) sin bloqueo

#### Scenario: Monto fuera de rango bloquea el avance
- **WHEN** el pagador ingresa un monto menor a S/ 10 o mayor a S/ 1000 en T3 e intenta continuar
- **THEN** el sistema bloquea el avance a T4 antes de confirmar, mostrando un mensaje de bloqueo

### Requirement: Mensaje de bloqueo desde el catálogo centralizado
Cuando el monto esté fuera del rango de bajo monto, el sistema SHALL mostrar un mensaje de bloqueo
obtenido del catálogo de mensajes centralizado (HU01-S0), no hardcodeado en la pantalla.

#### Scenario: Mensaje de bloqueo proviene del catálogo
- **WHEN** el sistema bloquea el avance de T3 a T4 por un monto fuera de rango
- **THEN** el texto del mensaje de bloqueo se obtiene del catálogo de mensajes centralizado

### Requirement: Piso técnico de captura del campo numérico
El campo de captura numérica de monto SHALL rechazar a nivel de entrada cualquier valor menor a S/ 0.01,
de forma independiente a la validación de negocio del rango S/ 10–S/ 1000.

#### Scenario: Campo rechaza un valor por debajo del piso técnico
- **WHEN** el pagador intenta ingresar un valor menor a S/ 0.01 en el campo de captura de monto
- **THEN** el campo rechaza ese valor a nivel de entrada, independientemente de si S/ 0.01 también
  incumple el rango de negocio de S/ 10–S/ 1000

### Requirement: Validación de rango en el servidor como defensa en profundidad
Al orquestar `POST /payments`, el servidor SHALL validar nuevamente que el monto esté dentro del rango de
S/ 10 a S/ 1000, de forma independiente a la validación ya realizada en la app.

#### Scenario: Servidor rechaza un monto fuera de rango pese a la validación previa de la app
- **WHEN** un monto fuera del rango S/ 10–S/ 1000 llega a `POST /payments` a pesar de haber superado la
  validación de la app
- **THEN** el servidor rechaza la solicitud por estar fuera de rango

### Requirement: Mensaje consistente ante discrepancia cliente-servidor
Cuando el servidor rechace un monto por estar fuera de rango tras haber pasado la validación de la app, el
sistema SHALL mostrar al pagador el mismo mensaje de bloqueo claro usado en la validación de la app, sin
dejarlo en un estado "Procesando" falso.

#### Scenario: Discrepancia entre validación de la app y del servidor
- **WHEN** el servidor rechaza un monto que había pasado la validación de la app por estar fuera del rango
  S/ 10–S/ 1000
- **THEN** el pagador recibe el mismo mensaje de bloqueo claro mostrado por la validación de la app, sin
  llegar a un estado "Procesando" falso
