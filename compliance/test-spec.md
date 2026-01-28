# LYTOK Compliance Test Specification

Esta suite de pruebas define el comportamiento esperado para cualquier implementación de un parser de LYTOK.

## 📝 Metodología de Prueba

Para validar un parser, este debe ser capaz de procesar los archivos en `/valid` y fallar con errores controlados en los archivos de `/invalid`.

#### 1. Pruebas de Validez (`/valid`)

A diferencia de otros formatos, la validación de LYTOK no se realiza comparando contra archivos JSON (debido a la falta de soporte nativo para BigInt y Date en JSON estándar). El parser debe:

**Cotejo Nativo:** Comparar el objeto o arreglo resultante del parseo contra una estructura de datos nativa predefinida en el lenguaje de implementación.

**Integridad de Tipos:** Verificar que los valores de tipo BigInt y Date se recuperen con su precisión y tipo original, no como simples strings.

**Espacios:** Respetar los espacios en blanco preservados dentro de los delimitadores de texto, ya sean lso backticks para textos escapados o los delimitadores normales para textos no escapados.

**Auto-delimitación:** Procesar correctamente la omisión del separador | o $ tras bloques autodelimitados.

**Recursividad:** Reconstruir árboles de datos de profundidad arbitraria.

#### 2. Pruebas de Error (`/invalid`)

El parser debe arrojar una excepción o error si encuentra:

- **missing-header.ltk:** Falta el header (schema) en una estructura uniforme.

- **type-mismatch.ltk:** Un valor en modo simple que no coincide con el símbolo global (ej: # seguido de texto sin escapar).

- **malformed.ltk:** Brackets o llaves sin cerrar o falta del delimitador del header y la data.

### 📝 Requerimientos de Tipado

Cualquier parser debe soportar nativamente:

Integers y Floats (64 bits).

BigInt (128 bits).

Booleanos (`?T/?F`).

Dates (ISO 8601).
