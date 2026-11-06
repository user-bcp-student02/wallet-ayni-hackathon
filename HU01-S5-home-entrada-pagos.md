# HU01-S5 — Home: entrada a la funcionalidad de pagos

## Historia de usuario

Como pagador,
quiero ver mi saldo disponible en la pantalla de inicio y acceder desde ahí a iniciar un nuevo pago,
para comenzar el flujo de transferencia sin fricción.

## Origen y trazabilidad

- Slice derivada de **HU01 — Confirmación de pagos de bajo monto** ([HU01-payment.md](./HU01-payment.md)).
- Cubre íntegramente **T1 (Home)**, antes incluido como parte del camino feliz en **HU01-S1**.
- Técnica de slicing aplicada: *workflow steps* — se aísla el punto de entrada (Home) del resto del flujo
  de pago, para poder construirlo, revisarlo y desplegarlo como el enlace/puerta de acceso a la
  funcionalidad de pagos, independientemente de la orquestación que viene después.
- **Impacto en el backlog existente:** esta slice reemplaza el contenido de T1 que estaba duplicado en
  **HU01-S1**; esa HU se actualizó para depender de esta en vez de describir Home por su cuenta (ver nota
  en HU01-S1).

## Alcance

- Saludo "Bienvenido, {Usuario}".
- Saldo disponible, obtenido con `GET /wallets/{id}`.
- Una acción principal y grande: **Nuevo pago**, que navega hacia la pantalla de selección de contacto
  (T2), cubierta por **HU01-S1**.
- Pantalla construida sobre los componentes y el catálogo de mensajes de **HU01-S0**.

## Criterios de aceptación

1. **Dado** un pagador autenticado, **cuando** entra a Home, **entonces** ve el saludo
   "Bienvenido, {Usuario}".
2. **Dado** que Home carga, **cuando** consulta el saldo, **entonces** se muestra el saldo disponible
   obtenido de `GET /wallets/{id}`.
3. **Dado** que el pagador quiere iniciar una transferencia, **cuando** presiona el botón **Nuevo pago**,
   **entonces** es dirigido a la pantalla de selección de contacto (T2) del flujo de pago.
4. **Dado** la pantalla Home, **cuando** se renderiza, **entonces** no ofrece ninguna otra acción principal
   además de **Nuevo pago** (el botón "Ver historial" permanece en backlog).

## Dependencias

- Depende de **HU01-S0** (componentes y catálogo de mensajes).
- Es prerrequisito de **HU01-S1**: el flujo de pago empieza donde termina la navegación de esta HU
  (selección de contacto, T2 en adelante).

## Endpoints referenciados

| Endpoint | Uso |
|---|---|
| `GET /wallets/{id}` | Saldo disponible en Home |

## Fuera de alcance de esta porción

- Botón "Ver historial" en Home: backlog de la HU original, sin cambios.
- Cualquier pantalla o lógica a partir de la selección de contacto (T2 en adelante): cubierta por
  **HU01-S1**.

## Preguntas abiertas (nuevas, no heredadas de HU01)

- La HU original no especifica el comportamiento de Home ante un error al consultar
  `GET /wallets/{id}` (p. ej. falla de red). No se infiere un comportamiento: queda pendiente de
  definición antes de implementar esta HU.

## Pruebas asociadas

- Carga de Home con saldo disponible (camino feliz).
- Navegación desde "Nuevo pago" hacia la selección de contacto.
