# Tasks

## 1. Validación de rango en el cliente (T3)

- [ ] 1.1 Implementar la regla de bloqueo de "Continuar" en T3 cuando el monto sea menor a S/ 10 o mayor a
      S/ 1000, y verificar con pruebas unitarias los casos límite S/ 10, S/ 9.99, S/ 1000 y S/ 1000.01
      (requirement "Rango de negocio permitido en la app").
- [ ] 1.2 Conectar el mensaje de bloqueo al catálogo de mensajes centralizado de HU01-S0 (sin texto
      hardcodeado) y verificar con una prueba que el mensaje mostrado proviene del catálogo (requirement
      "Mensaje de bloqueo desde el catálogo centralizado").
- [ ] 1.3 Configurar el campo de captura numérica para rechazar a nivel de entrada valores menores a
      S/ 0.01, y verificar con una prueba que el valor S/ 0.01 es el mínimo aceptado por el campo,
      independientemente del rango de negocio (requirement "Piso técnico de captura del campo numérico").

## 2. Validación de rango en el servidor (defensa en profundidad)

- [ ] 2.1 Agregar la validación de rango S/ 10–S/ 1000 al orquestar `POST /payments`, y verificar con una
      prueba de caracterización que un monto fuera de rango es rechazado por el servidor aunque se invoque
      directamente (sin pasar por la validación de la app) (requirement "Validación de rango en el
      servidor como defensa en profundidad").
- [ ] 2.2 Mapear el rechazo del servidor al mismo mensaje de bloqueo del catálogo de HU01-S0 y verificar
      con una prueba que, ante ese rechazo, la app muestra el mensaje sin dejar un estado "Procesando"
      visible (requirement "Mensaje consistente ante discrepancia cliente-servidor").

## 3. Pruebas de integración de extremo a extremo

- [ ] 3.1 Agregar una prueba de integración que simule un monto fuera de rango forzado a pasar la
      validación de la app (p. ej. manipulando la solicitud) y confirme que el flujo completo termina en
      el mensaje de bloqueo, no en un pago "Procesando" o "Exitoso" (caso de discrepancia cliente-servidor
      de la HU).
- [ ] 3.2 Ejecutar la suite de pruebas de `web/` y `api/` afectada por esta slice y verificar que la
      cobertura del código modificado alcanza 85% o más (meta del contrato de agentes).

## Workflow follow-up

- Revisión y aprobación humanas de los cambios de validación antes de integrarlos.
- Archivar este change una vez implementado y aprobado, siguiendo el flujo estándar de OpenSpec.
