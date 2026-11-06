# Spec Delta

## Purpose

Permite a un pagador autenticado ver su saldo disponible al entrar a la aplicación y acceder desde ahí a
iniciar un nuevo pago, como único punto de entrada a la funcionalidad de pagos.

## ADDED Requirements

### Requirement: Saludo al pagador autenticado
El sistema SHALL mostrar, al entrar a Home un pagador autenticado, el saludo "Bienvenido, {Usuario}" con el
nombre del pagador autenticado.

#### Scenario: Pagador autenticado ve el saludo de bienvenida
- **WHEN** un pagador autenticado entra a Home
- **THEN** ve el saludo "Bienvenido, {Usuario}" con su propio nombre de usuario

### Requirement: Saldo disponible obtenido de `GET /wallets/{id}`
El sistema SHALL consultar y mostrar en Home el saldo disponible del pagador, obtenido de
`GET /wallets/{id}`.

#### Scenario: Home muestra el saldo disponible al cargar
- **WHEN** Home carga para un pagador autenticado
- **THEN** consulta `GET /wallets/{id}` y muestra el saldo disponible devuelto

### Requirement: Única acción principal "Nuevo pago"
El sistema SHALL ofrecer en Home una única acción principal, **Nuevo pago**, que SHALL navegar hacia la
pantalla de selección de contacto (T2) del flujo de pago, y SHALL no ofrecer en Home ninguna otra acción
principal.

#### Scenario: Nuevo pago navega a selección de contacto
- **WHEN** el pagador presiona el botón **Nuevo pago** en Home
- **THEN** es dirigido a la pantalla de selección de contacto (T2) del flujo de pago

#### Scenario: Home no ofrece otra acción principal
- **WHEN** Home se renderiza para un pagador autenticado
- **THEN** no se muestra ninguna otra acción principal además de **Nuevo pago** (el botón "Ver historial"
  permanece en backlog)
