# Tasks

Alcance: `web/` únicamente (ver `proposal.md`). Ejecutar pruebas con `python tools/lab.py test web` desde la
raíz, o `npm test` / `npm run test:coverage` desde `web/`. No se modifican `LoginComponent`,
`DashboardComponent` ni los contratos `/auth/login` y `/api/tasks`.

## 1. Estructura del módulo compartido

- [ ] 1.1 Crear el directorio `web/src/app/payment-ui/` con subcarpetas `messages/` y `components/` (ver
  `design.md` - Decisions), sin modificar `login/` ni `dashboard/`. Verificar ejecutando
  `npm test` desde `web/` y confirmando que la suite existente sigue en verde (sin romper nada por la
  nueva carpeta vacía).

## 2. Catálogo de mensajes (`payment-ui-message-catalog`)

- [ ] 2.1 Implementar el servicio de catálogo (p. ej. `MessageCatalogService`) en `payment-ui/messages/` con
  un método de resolución por clave y una fuente de configuración editable (claves por categoría: saludo,
  resumen de confirmación, estados terminales, ayuda, errores), incluyendo la clave del mensaje
  tranquilizador de "En proceso". Verificar con una prueba unitaria que resuelve al menos una clave real por
  categoría y obtiene el texto configurado.
- [ ] 2.2 Implementar el comportamiento ante clave no configurada (retorno identificable como "mensaje no
  configurado", sin excepción no controlada). Verificar con una prueba unitaria que solicita una clave
  inexistente y comprueba el resultado identificable.
- [ ] 2.3 Verificar que editar el texto de una clave existente en la fuente de configuración no requiere
  cambios en ningún componente consumidor. Verificar con una prueba unitaria que cambia el valor de una
  clave en la configuración de prueba y confirma que el servicio devuelve el nuevo valor sin tocar otros
  archivos.

## 3. Componentes de UI base accesibles (`payment-ui-accessible-components`)

- [ ] 3.1 Implementar el componente de texto base en `payment-ui/components/` que aplique tamaño ≥ 18 px y
  contraste WCAG AA vía CSS propio. Verificar con una prueba de componente que inspecciona el tamaño de
  fuente y el contraste calculado (o las variables/clases CSS que lo garantizan) del elemento renderizado.
- [ ] 3.2 Implementar el componente de botón base con área táctil ≥ 48 px y rol/label accesibles. Verificar
  con una prueba de componente que mide las dimensiones renderizadas y confirma el atributo de accesibilidad
  (`role`/`aria-label`) presente.
- [ ] 3.3 Implementar el componente de ícono+texto base sin modo "solo ícono" (API que siempre exige un
  texto asociado). Verificar con una prueba de componente que confirma que el texto es obligatorio
  (falla o se omite el render si no se provee) y que el ícono se marca `aria-hidden` mientras el texto es
  anunciado.
- [ ] 3.4 Implementar el componente de layout base de pantalla con un único punto de contenido principal y
  el elemento de ayuda visible ("¿Necesitas ayuda?") mostrado por defecto. Verificar con una prueba de
  componente que renderiza el layout sin configuración adicional y confirma la presencia del elemento de
  ayuda.
- [ ] 3.5 Implementar el componente base de estado "En proceso", que resuelve su mensaje desde el catálogo
  de mensajes (tarea 2.1) y persiste su bandera de "en curso" en almacenamiento local del navegador (ver
  `design.md` - Decisions). Verificar con una prueba de componente que simula recarga (reinstanciar el
  componente con el mismo almacenamiento local) y confirma que el mensaje tranquilizador se vuelve a
  mostrar.
- [ ] 3.6 Verificar compatibilidad con lector de pantalla de los cinco componentes base (texto, botón,
  ícono+texto, layout, estado "En proceso"): cada uno expone roles/labels accesibles. Verificar con pruebas
  de componente que consultan los atributos ARIA renderizados de cada uno.

## 4. Verificación y evidencia

- [ ] 4.1 Ejecutar `npm run test:coverage` desde `web/` (o `python tools/lab.py test web` desde la raíz) y
  confirmar que toda la suite (existente + nueva) pasa en verde.
- [ ] 4.2 Confirmar que la cobertura del código modificado/agregado en `payment-ui/` alcanza ≥ 85% según el
  reporte de `test:coverage`.
- [ ] 4.3 Registrar en el documento general de hallazgos la sección `web`, con el resultado de las pruebas y
  la cobertura obtenida para esta slice (HU01-S0), sin tocar otras secciones del documento.

## Fuera de alcance (no incluir en estas tareas)

- Construcción de las pantallas funcionales del flujo de pago (saludo, confirmación, resultados,
  orquestador): corresponde a HU01-S1–S4.
- Copy final de los mensajes (etiquetas de estados terminales, textos de ayuda definitivos): pendiente con
  producto.
- Persistencia real del intento de pago en backend: corresponde a la integración de HU01-S1.
- Cambios en `auth/` o `api/`, en `LoginComponent`, `DashboardComponent`, o en los contratos `/auth/login` y
  `/api/tasks`.
