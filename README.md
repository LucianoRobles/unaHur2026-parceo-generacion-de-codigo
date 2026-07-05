# CookLang

**CookLang** es un lenguaje de programación simple diseñado para describir recetas de cocina de forma estructurada.

El objetivo del proyecto es definir un lenguaje propio y luego implementar sus componentes principales: scanner, parser, acciones semánticas y casos de prueba automatizados.

## Descripción del lenguaje

CookLang permite escribir recetas utilizando instrucciones claras para:

- Definir una receta.
- Agregar ingredientes.
- Mezclar ingredientes.
- Cocinar una preparación.
- Enfriar una preparación.
- Indicar cantidad de porciones.
- Utilizar una estructura condicional `If then else`.

Ejemplo:

```text
Recipe "Tarta de manzana" {
    Add 200g harina;
    Add 100g azucar;
    Add 2pcs manzanas;
    Mix harina azucar manzanas;
    If tiene_horno then Bake 45 minutes else Chill 60 minutes;
    Serve 4 portions;
}
```

## Componentes del trabajo

El proyecto se organiza en las siguientes etapas:

1. Definición y descripción del lenguaje — ✅ `cooklang_lenguaje.md`
2. Implementación del scanner — ✅ `cooklang_scanner.md`
3. Implementación del parser — ✅ `cooklang_parser.md`
4. Implementación de acciones semánticas y traducción a JSON — ✅ `cooklang_semantica.md`
5. Casos de prueba automatizados — ✅ `cooklang_automation_test.ipynb`
6. Conclusiones — ✅ `cada documento incluye su propia conclusión al final de la etapa correspondiente.`

## Estructura del repositorio

| Archivo | Contenido |
|---|---|
| `cooklang_lenguaje.md` | Definición formal del lenguaje: objetivo, alcance, especificación léxica, sintáctica y semántica. |
| `cooklang_scanner.md` | Tabla de lexemas, autómata de transiciones y su implementación con `ply.lex`. |
| `cooklang_parser.md` | Gramática independiente del contexto, tabla de parseo (ASDB) e implementación con `ply.yacc`. |
| `cooklang_semantica.md` | Validaciones semánticas y traducción de una receta a JSON. |
| `cooklang_automation_test.ipynb` | Notebook ejecutable con el pipeline completo (scanner + parser + semántica) y casos de prueba: programa válido, error léxico, error sintáctico y error semántico. |

## Scanner

El scanner reconoce los tokens principales del lenguaje, como palabras reservadas, identificadores, números enteros, cadenas de texto, unidades de medida y símbolos especiales.

Algunos tokens reconocidos son:

```text
RECIPE
ADD
MIX
BAKE
CHILL
IF
THEN
ELSE
SERVE
ID
INT
STRING
UNIT
LLAVE_ABRE
LLAVE_CIERRA
PYC
```

## Parser y acciones semánticas

El parser valida que la secuencia de tokens respete la gramática de CookLang (`cooklang_parser.md`).

Sobre esa base, las acciones semánticas verifican reglas que la gramática por sí sola no puede expresar cantidades y tiempos positivos, ingredientes declarados antes de usarse, ingredientes no repetidos y traducen la receta a una estructura JSON (`cooklang_semantica.md`).

Ejemplo de traducción:

```json
{
  "recipe": "Tarta de manzana",
  "ingredients": [
    { "name": "harina", "quantity": 200, "unit": "g" },
    { "name": "azucar", "quantity": 100, "unit": "g" },
    { "name": "manzanas", "quantity": 2, "unit": "pcs" }
  ],
  "steps": [
    { "action": "mix", "ingredients": ["harina", "azucar", "manzanas"] },
    {
      "condition": "tiene_horno",
      "then": { "action": "bake", "time": 45, "unit": "minutes" },
      "else": { "action": "chill", "time": 60, "unit": "minutes" }
    }
  ],
  "servings": 4
}
```

## Cómo ejecutar el notebook

El pipeline completo (scanner, parser, acciones semánticas y casos de prueba) está en `cooklang_automation_test.ipynb`, pensado para correr en Jupyter.

1. Instalar la dependencia: `pip install ply` (primera celda del notebook).
2. Ejecutar las celdas en orden, de arriba hacia abajo.
3. La celda de imports fija un `__file__` dummy en el módulo `__main__` es necesario porque PLY inspecciona el archivo fuente del módulo llamador para validar reglas, y en Jupyter `__main__` no tiene `__file__` propio.
4. El parser se construye con `yacc.yacc(write_tables=False, debug=False)` para no depender de archivos de tabla (`parsetab.py`, `parser.out`) cacheados en disco entre corridas.

## Estado del proyecto

El lenguaje está definido y las tres etapas de análisis (léxico, sintáctico y semántico) están implementadas y documentadas, junto con la traducción de una receta a JSON.

El notebook `cooklang_automation_test.ipynb` ejecuta el pipeline de punta a punta y cubre casos de prueba automatizados: un programa válido y un caso de error por cada etapa (léxico, sintáctico y semántico).
