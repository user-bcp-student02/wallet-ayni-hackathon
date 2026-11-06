# Tasks

Alcance: componente de presentación de resultado de `web/` para HU01-S6 (capability
`payment-result-presentation`). No incluye el punto de invocación real (orquestación de HU01-S1/S4 ni
mapeo de errores de HU01-S3): este componente se construye con una interfaz de entrada fija (estado +
datos) y se prueba de forma aislada, reutilizando los componentes base y el catálogo de mensajes de
HU01-S0.

## 1. Estructura del componente y estado Exitoso

- [ ] 1.1 Crear el componente de presentación de resultado bajo `web/src/app/` (estructura de carpetas a
      definir junto con el resto de `web/src/app`), que acepta como entrada un estado
      (Procesando/Exitoso/Fallido/Expirado) y los datos del pago (`id`, fecha, destino, monto).
      Verificar con una prueba de componente que renderiza sin errores dado un estado Exitoso de ejemplo.
- [ ] 1.2 Implementar el renderizado del estado Exitoso: los cuatro campos (`id`, fecha, destino, monto) y
      el mensaje de éxito correspondiente del catálogo de mensajes de HU01-S0. Verificar con una prueba de
      componente que confirma los cuatro campos y el mensaje de éxito en el DOM renderizado.

## 2. Estados Fallido y Expirado

- [ ] 2.1 Implementar el renderizado del estado Fallido reutilizando el mismo layout de campos (`id`,
      fecha, destino, monto) del estado Exitoso, con el mensaje de Fallido del catálogo de mensajes.
      Verificar con una prueba de componente que confirma los cuatro campos y el mensaje de Fallido.
- [ ] 2.2 Implementar el renderizado del estado Expirado con el mismo layout de campos y el mensaje de
      Expirado del catálogo de mensajes. Verificar con una prueba de componente que confirma los cuatro
      campos y el mensaje de Expirado.

## 3. Estado Procesando y persistencia del intento

- [ ] 3.1 Implementar el renderizado del estado "Procesando", mostrando el mensaje tranquilizador del
      catálogo de mensajes en lugar de los datos finales del pago. Verificar con una prueba de componente
      que confirma el mensaje tranquilizador y la ausencia de los campos finales mientras el estado es
      "Procesando".
- [ ] 3.2 Implementar la persistencia del intento en estado "Procesando" para que, si el pagador cierra y
      reabre la app antes de alcanzar un estado terminal, la pantalla vuelva a mostrar "Procesando" para
      ese mismo intento (mecanismo concreto de almacenamiento a decidir en esta tarea). Verificar con una
      prueba que simule cerrar/reabrir (recarga del estado del componente) y confirme que "Procesando"
      sigue visible para el mismo intento.

## 4. Navegación "Volver al inicio"

- [ ] 4.1 Implementar la acción **Volver al inicio**, disponible en los tres estados terminales (Exitoso,
      Fallido, Expirado), que navega a Home (HU01-S5). Verificar con una prueba de navegación/routing para
      cada uno de los tres estados terminales.

## 5. Frontera de responsabilidad (pantalla tonta)

- [ ] 5.1 Confirmar, con una prueba que inspeccione las dependencias del componente (o un espía sobre el
      cliente HTTP usado en las pruebas), que el componente no emite ninguna solicitud propia a
      `POST /payments` ni a `GET /payments/{id}`, y que no contiene lógica de mapeo de códigos de error
      HTTP.

## 6. Verificación general

- [ ] 6.1 Confirmar que la cobertura del código modificado en `web/` para este componente alcanza ≥ 85%.
      Verificar con el reporte de cobertura de `python tools/lab.py test web` ejecutado desde la raíz.
- [ ] 6.2 Registrar evidencia en el documento general de hallazgos (sección `web`, HU01-S6) con el
      resultado de las pruebas y la cobertura obtenida, conservando las secciones existentes del
      documento.

## Workflow follow-up

- Antes de que HU01-S1, HU01-S3 o HU01-S4 integren este componente en su propio punto de invocación,
  confirmar que la interfaz de entrada (estado + `id`/fecha/destino/monto) implementada aquí coincide con
  lo que esas HU ya asumen en sus propias specs/diseños.
- Archivar este change una vez aprobada la implementación, con revisión humana y pruebas reales
  ejecutadas (no inventadas), según el contrato general de agentes del repositorio.
