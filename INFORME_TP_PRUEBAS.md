# Informe del Trabajo Practico - Calidad y Pruebas de Software

## 1. Resumen

Se resolvio el laboratorio introductorio de pruebas automatizadas con Express.js y TypeScript.

Los objetivos principales cumplidos fueron:

- Diagnosticar la falla intencional del descuento superior a 100 %.
- Incorporar validaciones al calculo de precios.
- Completar las pruebas unitarias exigidas.
- Agregar una prueba de integracion HTTP.
- Verificar tipos con TypeScript.
- Preparar la integracion continua para instalaciones reproducibles.

## 2. Diagnostico inicial

Antes de realizar los cambios, la ejecucion de `npm test` produjo:

- 4 pruebas aprobadas.
- 1 prueba fallida.
- 2 archivos de pruebas ejecutados.

La prueba fallida era:

```typescript
expect(() => calcularPrecioFinal(1000, 150)).toThrow(RangeError);
```

La implementacion original devolvia `-500`, porque calculaba:

```text
1000 - (1000 * 150) / 100 = -500
```

Este resultado no cumplia la regla de negocio, ya que un descuento no puede superar el 100 %. Ademas, la funcion no lanzaba el `RangeError` esperado por Vitest.

## 3. Cambios realizados

### 3.1 Validacion del servicio de precios

Se modifico `demo_practica/src/services/precios.ts`.

Se incorporaron las siguientes validaciones:

1. El precio debe ser un numero finito.
2. El precio debe ser igual o mayor que cero.
3. El descuento debe ser un numero finito.
4. El descuento debe estar entre 0 y 100, inclusive.
5. Las entradas invalidas lanzan `RangeError`.

Mensajes utilizados:

- `"El precio debe ser un numero finito igual o mayor que cero"`
- `"El descuento debe estar entre 0 y 100"`

Se conserva la formula original para entradas validas:

```typescript
precio - (precio * descuento) / 100
```

### 3.2 Compatibilidad con ambos endpoints de salud

Se modifico `demo_practica/src/app.ts` para conservar la ruta usada por el laboratorio actualizado:

- `GET /salud`

Ademas, se agrego el alias:

- `GET /healthy`

Ambas rutas devuelven:

```json
{"estado":"ok"}
```

### 3.3 Pruebas unitarias agregadas

Se modifico `demo_practica/tests/precios.test.ts`.

Se agregaron pruebas para:

- Descuento del 100 %, con resultado esperado `0`.
- Precio negativo, que debe lanzar `RangeError`.
- Descuento negativo, que debe lanzar `RangeError`.
- Precio `NaN`, que debe lanzar `RangeError`.

Tambien se conservaron las pruebas originales:

- Descuento del 10 %, con resultado `900`.
- Descuento del 0 %, con resultado igual al precio.
- Descuento del 150 %, que debe lanzar `RangeError`.

### 3.4 Prueba de integracion HTTP agregada

Se modifico `demo_practica/tests/api.test.ts`.

Se agrego una prueba que envia una solicitud HTTP:

- Metodo: `POST`
- Ruta: `/api/precios/calcular`
- Cuerpo: `{"precio":1000,"descuento":150}`
- Estado esperado: `400`
- Propiedad esperada en el cuerpo: `error`

Esta prueba es de integracion porque verifica de manera conjunta:

- El manejo HTTP de Express.
- El parseo del cuerpo JSON.
- La ruta de la API.
- La funcion `calcularPrecioFinal`.
- La conversion de `RangeError` en una respuesta HTTP 400.

### 3.5 Integracion continua

Se modifico `demo_practica/.github/workflows/pruebas.yml`.

El paso de instalacion cambio de:

```yaml
- run: npm install
```

a:

```yaml
- run: npm ci
```

Esto utiliza el archivo `package-lock.json` para realizar instalaciones reproducibles.

### 3.6 Dependencias bloqueadas

Se genero `demo_practica/package-lock.json` mediante `npm install`.

Este archivo permite que GitHub Actions utilice `npm ci` y fija las versiones resueltas del proyecto.

## 4. Comprobaciones ejecutadas

### 4.1 Pruebas automatizadas

Comando:

```powershell
npm test
```

Resultado final:

- 2 archivos de pruebas aprobados.
- 10 pruebas aprobadas.
- 0 pruebas fallidas.
- Vitest finalizo con resultado exitoso.

### 4.2 Verificacion de tipos

Comando:

```powershell
npm run typecheck
```

Resultado:

- `tsc --noEmit` finalizo sin errores.

### 4.3 Verificacion manual de endpoints

Se inicio la API y se verificaron:

- `GET /salud`: `{"estado":"ok"}`
- `GET /healthy`: `{"estado":"ok"}`
- `POST /api/precios/calcular` con descuento 150: estado HTTP `400`

Respuesta recibida:

```json
{"error":"El descuento debe estar entre 0 y 100"}
```

## 5. Tabla de resultados del laboratorio

| Comprobacion | Resultado |
|---|---|
| Respuesta de `GET /salud` | Exitosa |
| Respuesta de `GET /healthy` | Exitosa |
| Primera ejecucion de `npm test` | 4 aprobadas y 1 fallida |
| Correccion del servicio | Aplicada |
| Nuevas pruebas unitarias | 7 aprobadas |
| Prueba de integracion HTTP 400 | Aprobada |
| Verificacion `npm run typecheck` | Exitosa |
| Resultado total de Vitest | 10 pruebas aprobadas |

## 6. Respuestas a las preguntas

### 1. ¿Que diferencia hay entre comprobar manualmente una respuesta y ejecutar una prueba automatizada?

Una comprobacion manual depende de una persona y puede tener mayor riesgo de omision o variacion. Una prueba automatizada expresa la expectativa en codigo y puede repetirse siempre con el mismo criterio.

### 2. ¿Cual fue la causa de la primera prueba fallida?

La funcion no validaba el rango del descuento. Por eso calculaba un total negativo con el 150 % en lugar de lanzar `RangeError`.

### 3. ¿Por que es util incluir casos invalidos?

Los casos invalidos verifican que el sistema tambien gestione datos incorrectos o peligrosos. Los casos habituales no alcanzan para comprobar limites, precios negativos, descuentos fuera de rango o valores no finitos.

### 4. ¿Que parte verifica cada archivo de pruebas?

- `tests/precios.test.ts`: verifica de manera aislada la funcion de calculo.
- `tests/api.test.ts`: verifica la colaboracion entre Express, la ruta HTTP y el servicio de precios.

### 5. ¿Que ventajas ofrece ejecutar las pruebas despues de cada modificacion?

- Permite detectar regresiones.
- Entrega una respuesta rapida sobre el impacto de los cambios.
- Convierte los requisitos en comprobaciones repetibles.
- Facilita corregir errores antes de integrar el codigo.
- Puede automatizarse desde GitHub Actions.

### 6. ¿Por que una respuesta HTTP 200 puede contener errores de negocio?

HTTP 200 indica que una solicitud fue procesada correctamente desde el punto de vista del protocolo. No garantiza que los datos de negocio sean validos. Por eso es necesario comprobar tambien el contenido de la respuesta y las reglas del dominio.

### 7. ¿Por que `toThrow` recibe una funcion?

Vitest necesita ejecutar la funcion dentro de su propia comprobacion para capturar la excepcion. Si se pasa directamente el resultado, la funcion se ejecuta antes de que `toThrow` pueda inspeccionar el lanzamiento del error.

### 8. ¿Que diferencia hay entre `npm test` y `npm run typecheck`?

- `npm test` ejecuta el comportamiento definido en las pruebas.
- `npm run typecheck` analiza el codigo y sus tipos sin ejecutar las pruebas ni transformar JavaScript.

Ambas verificaciones son complementarias.

### 9. ¿Que aporta GitHub Actions al trabajo colaborativo?

GitHub Actions ejecuta automaticamente las verificaciones despues de cada `push` o `pull request`. Esto permite detectar fallos antes de integrar cambios y proporciona un resultado visible y repetible para todo el equipo.

## 7. Archivos modificados o creados

Archivos modificados:

- `demo_practica/src/services/precios.ts`
- `demo_practica/src/app.ts`
- `demo_practica/tests/precios.test.ts`
- `demo_practica/tests/api.test.ts`
- `demo_practica/.github/workflows/pruebas.yml`

Archivo creado:

- `demo_practica/package-lock.json`


