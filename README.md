<p align="center">
<img src="assets/icon/codnn-logo-256x256.png" alt="CODNN Logo" width="200">
</p>

<p align="center">
<img src="https://img.shields.io/badge/version-1.0.0--beta-blue" alt="Version">
<img src="https://img.shields.io/badge/license-Apache 2.0-orange" alt="License">
<img src="https://img.shields.io/badge/format-.odn-purple" alt="Format">
</p>

# CODNN Specification (Open Core)

**CODNN** (Compact Object Data Native Notation) es un estándar de serialización híbrido diseñado para optimizar el intercambio de información en entornos de alta carga y aplicaciones de Inteligencia Artificial.

El formato utiliza la extensión oficial .odn (Open Data Notation).

---

### 💡 ¿Por qué existe CODNN?

El mundo se mueve sobre **JSON**, un formato excelente para la legibilidad humana pero costoso para el procesamiento de máquinas y modelos de lenguaje (LLMs). En la era de la IA, cada carácter cuenta:

- **Gasto de Contexto:** JSON repite las llaves de los objetos en cada elemento de un arreglo, desperdiciando espacio vital en la ventana de contexto de los modelos.

- **Ambigüedad de Tipos:** JSON no diferencia nativamente entre un entero y un flotante, ni soporta fechas o BigInts sin transformaciones adicionales que degradan el rendimiento.

CODNN nace para ser la "destilación" de los datos: mantiene la estructura pero elimina el ruido.

### 🏗️ Modos de Operación y Sintaxis

La sintaxis de CODNN cambia según la homogeneidad de los datos para maximizar el ahorro de espacio.

#### 1. Arreglos Uniformes (#Schema).

Es el modo de máxima eficiencia. El esquema (Header) define el orden posicional.

**Header con conteo:** `#User[2]:id|nombre::` indica que siguen 2 registros.

**Tipado en Schema:** Las propiedades pueden llevar un símbolo de tipo: `edad#|activo?|`. Si no tiene símbolo, se asume **String**.

#### 2. Arreglos Simples (#, ?, @, &, `)

Para listas de un solo tipo de dato primitivo.

- **Ejemplo (Números):** #1;2;3;4

- **Ejemplo (Fechas):** @2025-01-01:00:00:00Z;2025-02-01:00:00:00Z

#### 3. Arreglos Mixtos (\*)

Para datos heterogéneos. Cada valor debe llevar su prefijo de tipo.

**Ejemplo:** `\*#123;?T;texto;^` (Donde `^` es Null).

### 💎 Tipos de Datos Avanzados

##### CODNN soporta nativamente tipos que otros formatos de texto ignoran, garantizando cero pérdida de precisión:

| Símbolo |  Tipo   |                                Regla Técnica                                |
| :-----: | :-----: | :-------------------------------------------------------------------------: |
|    #    | Number  | Soporta Integers y Floats (f64) detectados por la presencia del punto `.`.  |
|    &    | BigInt  |              Soporta enteros de hasta 128 bits sin redondeos.               |
|    ?    | Boolean |                  Representado por ?T (True) o ?F (False).                   |
|    @    |  Date   | Obligatorio formato ISO 8601 (YYYY-MM-DDTHH:mm:ss.sssZ) para parseo nativo. |
|    `    |  Text   |  Encapsulamiento con backticks. Los backticks internos se escapan como ``.  |
|    ^    |  Null   |                      Representa la ausencia de valor.                       |

### 📏 Reglas de Indentación y Formato

Cuando se utiliza el modo **Formatted** (`formatted: true`), se aplican las siguientes reglas para mantener la legibilidad:

**1. Generalidad:**

- **Indentación:** 4 espacios por nivel (formatted).

- **Recursividad:** Soporte total de mapas dentro de arreglos y viceversa.

- **Manejo de espacios:** En los textos, los espacios dentro de los delimitadores son respetados de forma nativa.

- **Registros raíz:** El esquema uniforme indica la cantidad total: `#lote[5]`.

- **Conteo en arreglos anidados:** Inician con el número de registros y un delimitador: `[2|`. El número está al nivel del bracket de apertura, mientras que los datos siguen la indentación.

**2. Separadores Estructurales:**

- `|` : Separador de campos en Mapas/Objetos.

- `\n-\n` : Separador de registros en Arreglos de nivel superior para arreflos uniformes (formatted).

- `;` : Separador de elementos en Arreglos de nivel superior para arreglos simples/mixtos(minified y formatted) y uniformes (minified).

- `$` : Separador de elementos en Arreglos anidados.

**3. Auto-delimitación:**

Ciertos valores permiten omitir el separador estructural (| o $) para ahorrar espacio:

- **Strings (\`texto\`):** Son aquellos encerrados entre ` `` `, al estar encapsulados, el formato puede omitir el separador de estructural (`|` o `$`) según sea el caso.

- **Bloque (`{}` `[]`):** Al ser delimitadores que marcan el inicio o fin de un objeto/arreglo, estos permiten la omision del separador estructural.

- **Nulidad (`^`):** Al ser un simbolo que refleja un valor nulo directo, no es necesario estar delimitado, cuando el interpretador detecte el simbolo, regresa de forma inmediata un nulo.

- **Excepción:** Si hay dos campos de texto escapados consecutivos, el delimitador `|` es obligatorio.

#### Ejemplo Formateado Complejo:

```Text
#lote[2]:
  id|fecha|stock_total#|es_importado?|es_certificado?|fabricante{
    nombre|registro|direccion{
      calle|ciudad|zip#
    }
  }|datosLaboratorio{
    fecha_prueba|direccion{
      calle|edificio|referencia
    }
  }|certificaciones[
    nombre_cert|valida_hasta
  ]|sucursales[
    nombre|ciudad|empleados[
      nombre|apellido|datos{
        calle|numero#|telefono#|correo
      }
    ]
  ]::
`L|001`2025-11-20|50000|F|T{
    Acme Corp|0123456789{
        Calle Falsa 123|Springfield|62000
    }
}^[2|
    ISO-9001|2026-01-01$CQC-A|2027-05-15
][2|
    suc-52|La Paz[2|
        Maria|Perez{
            calle imaginaria|520|6489635672|maria.perez@mail.com
        }$`Eusebio;`Mendez{
            calle imaginaria sur|580|6489635652|eusebio.m@mail.com
        }
    ]$suc-105|Tepito^
]
-
L002|2025-11-20|15000|T|F{
    Industrias Z|987654321{
        Av. Siempre Viva 742|CDMX|90210
    }
}{
    2025-11-25{
        Rio Tiber|Torre Central|2ndo piso
    }
}[2|
    ISO-9001|2026-01-01$Cert_MX_IM|2026-01-01
]^
```

### 📂 Estructura del Proyecto

- `/spec`: Gramática formal EBNF que define las reglas de autodelimitación.

- `/compliance`: El Test Suite oficial para validar parsers de terceros.

- `/assets`: Identidad visual y diagramas de flujo de datos.

- `/examples`: Archivos .odn con casos de uso reales.

### SDK's disponibles

- #### [JS](https://github.com/Joguel96/codnn-js)

---

### 📜 Licencia

- #### Especificación (Gramática y Reglas): [Apache License 2.0](https://github.com/Joguel96/codnn-spec/blob/main/spec/LICENSE)

- #### Compliance Suite y Herramientas: [MIT License](https://github.com/Joguel96/codnn-spec/blob/main/compliance/LICENSE)

---

_Desarrollado por [joguel96](https://github.com/joguel96). CODNN es la respuesta a la necesidad de un transporte de datos más inteligente y ligero._
