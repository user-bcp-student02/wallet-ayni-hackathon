# HU01-S0 — Catálogo de mensajes y componentes de accesibilidad base

## Historia de usuario

Como equipo de desarrollo del flujo de pago,
quiero contar con un catálogo de mensajes centralizado/editable y un conjunto de componentes de UI
accesibles reutilizables,
para que todas las pantallas del flujo de pago cumplan los requisitos de inclusión sin duplicar lógica
ni copy.

## Origen y trazabilidad

- Slice derivada de **HU01 — Confirmación de pagos de bajo monto** ([HU01-payment.md](./HU01-payment.md)).
- Cubre íntegramente **T11 (Requisitos de inclusión)** y el criterio **CA5 — Inclusión y accesibilidad**,
  extraídos como habilitador técnico independiente (no como historia funcional de pagador) porque son un
  requisito no funcional transversal consumido por el resto de slices (S1–S4), no un flujo con valor propio
  para el pagador.
- Técnica de slicing aplicada: NFR como habilitador reutilizable en lugar de criterio repetido en cada
  historia.

## Alcance

- Catálogo de mensajes centralizado y editable (mecanismo, no el copy final) para los textos visibles en
  cada pantalla del flujo (saludo, resumen de confirmación, estados terminales, ayuda, errores).
- Componentes de UI base compartidos que garanticen:
  - Texto ≥ 18 px y contraste WCAG AA.
  - Botones grandes (≥ 48 px).
  - Íconos siempre acompañados de texto, nunca solos.
  - Un objetivo por pantalla y lenguaje simple, sin jerga.
  - Elemento de ayuda visible en cada pantalla ("¿Necesitas ayuda?").
  - Compatibilidad con lector de pantalla.
  - Estado "En proceso" con mensaje tranquilizador, sin perder el intento si se cierra la app.

## Criterios de aceptación

1. **Dado** que una pantalla del flujo de pago usa los componentes base provistos por esta slice,
   **cuando** se renderiza, **entonces** cumple automáticamente tamaño de texto, contraste, tamaño de
   botones e íconos con texto, sin que la pantalla implemente esa lógica por separado.
2. **Dado** cualquier texto visible al pagador, **cuando** se necesita mostrarlo, **entonces** se obtiene
   desde el catálogo de mensajes centralizado (no hardcodeado en el componente de pantalla).
3. **Dado** un lector de pantalla activo, **cuando** el pagador navega cualquier pantalla construida sobre
   estos componentes, **entonces** los elementos son anunciados correctamente (roles/labels accesibles).
4. **Dado** el estado "En proceso", **cuando** el pagador cierra y reabre la app, **entonces** el mensaje
   tranquilizador se mantiene y el intento de pago no se pierde (requisito de UI; la persistencia del
   intento en sí depende de la integración definida en HU01-S1).

## Dependencias

- No depende de ninguna otra slice: es el primer elemento del backlog derivado de HU01.
- Es prerrequisito de **HU01-S1, HU01-S2, HU01-S3 y HU01-S4**: todas construyen sus pantallas sobre estos
  componentes y ese catálogo.

## Fuera de alcance de esta porción

- Contenido final de las etiquetas de los estados terminales (Exitoso/Fallido/Expirado) y del resto del
  catálogo: se define el mecanismo editable, no el copy definitivo (pendiente con producto, ver
  **Preguntas abiertas**).
- Cualquier lógica de negocio de pago (orquestación, validación de monto, mapeo de errores): corresponde a
  las slices S1–S4.
- Pruebas de usabilidad con usuarios reales no nativos digitales (se ejecutan sobre el flujo integrado en
  HU01-S1, usando estos componentes como base).

## Supuestos heredados de HU01

- Las etiquetas visibles al usuario para los estados terminales quedan por definir con producto; mientras
  tanto se usa este catálogo de mensajes centralizado y editable.

## Preguntas abiertas heredadas de HU01

- Etiquetas finales de los estados terminales y contenido definitivo del catálogo de mensajes (T11).

## Pruebas asociadas (subconjunto de T12 original)

- Pruebas de usabilidad con usuarios no nativos digitales, ejecutadas contra los componentes base (la
  validación end-to-end se completa una vez integrada en HU01-S1).
