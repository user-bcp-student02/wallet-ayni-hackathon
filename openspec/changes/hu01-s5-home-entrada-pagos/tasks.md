# Tasks

Alcance: pantalla Home de `web/` para HU01-S5 (capability `payment-home-entry`). La disponibilidad de
`GET /wallets/{id}` (real o mock) y el comportamiento ante error son prerrequisitos externos: ver
`## Workflow follow-up`. Estas tareas no deciden ni implementan ese backend ni ese comportamiento de error.

## 1. Saludo y saldo disponible (T1) — `payment-home-entry`

- [ ] 1.1 Implementar la pantalla Home que muestra el saludo "Bienvenido, {Usuario}" con el nombre del
      pagador autenticado, reutilizando los componentes base y el catálogo de mensajes de HU01-S0.
      Verificar con una prueba de componente que confirma la interpolación correcta del nombre.
- [ ] 1.2 Implementar la consulta a `GET /wallets/{id}` al cargar Home y la presentación del saldo
      disponible devuelto. Verificar con una prueba de componente que, dado un mock de la respuesta,
      confirma que el saldo mostrado coincide con el valor devuelto.

## 2. Acción "Nuevo pago" (T1 → T2) — `payment-home-entry`

- [ ] 2.1 Implementar el botón **Nuevo pago** como única acción principal de Home, reutilizando los
      componentes base de HU01-S0. Verificar con una prueba de componente que confirma que no se renderiza
      ninguna otra acción principal junto a él.
- [ ] 2.2 Conectar la navegación del botón **Nuevo pago** hacia la pantalla de selección de contacto (T2)
      de HU01-S1. Verificar con una prueba de routing/navegación.

## 3. Verificación end-to-end

- [ ] 3.1 Ejecutar el escenario de carga de Home con saldo disponible seguido de la navegación desde
      "Nuevo pago" hacia la selección de contacto, de extremo a extremo, contra un backend real o mock
      disponible. Verificar con la prueba de integración/e2e de este escenario.
- [ ] 3.2 Confirmar que la cobertura del código modificado en `web/` para la pantalla Home alcanza ≥ 85%.
      Verificar con el reporte de cobertura de `python tools/lab.py test web` ejecutado desde la raíz.
- [ ] 3.3 Registrar evidencia en el documento general de hallazgos (sección `web`, HU01-S5) con el
      resultado de las pruebas y la cobertura obtenida, conservando las secciones existentes del
      documento.

## Workflow follow-up

- Antes de iniciar la tarea 1.2 (o en paralelo, si ya está resuelto), confirmar que `GET /wallets/{id}`
  está disponible — real o mock — con el contrato descrito en el Alcance de
  `HU01-S5-home-entrada-pagos.md`; si no lo está, resolverlo en un change separado antes de continuar (ver
  `design.md` — Open Questions).
- Definir con el equipo/negocio el comportamiento de Home ante un error al consultar
  `GET /wallets/{id}` (pendiente en la HU original); una vez definido, añadirlo como requisito a
  `specs/payment-home-entry/spec.md` antes de implementarlo.
- Archivar este change una vez aprobada la implementación, con revisión humana y pruebas reales
  ejecutadas (no inventadas), según el contrato general de agentes del repositorio.
