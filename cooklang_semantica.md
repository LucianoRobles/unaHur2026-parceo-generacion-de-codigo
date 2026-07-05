# Acciones semánticas de CookLang

Las acciones semánticas de **CookLang** se encargan de analizar el significado del programa una vez que ya fue aceptado sintácticamente por el parser.

Mientras que el parser verifica que las instrucciones estén escritas con la estructura correcta, las acciones semánticas verifican que esas instrucciones tengan sentido dentro de una receta.

Además, en esta etapa se construye una representación estructurada de la receta en formato JSON.

---

## Validaciones semánticas

Las principales validaciones semánticas realizadas por CookLang son:

| Validación                | Descripción                                                                      |
| ------------------------- | -------------------------------------------------------------------------------- |
| Cantidades positivas      | Las cantidades de ingredientes deben ser mayores que cero.                       |
| Tiempos positivos         | Los tiempos de cocción o enfriado deben ser mayores que cero.                    |
| Porciones positivas       | La cantidad de porciones debe ser mayor que cero.                                |
| Ingredientes declarados   | Todo ingrediente usado en `Mix` debe haber sido declarado previamente con `Add`. |
| Ingredientes no repetidos | No se puede declarar dos veces el mismo ingrediente con `Add`.                   |
| Nombre de receta limpio   | El nombre de la receta se almacena sin las comillas externas del token `STRING`. |

---

## Ejemplos de errores semánticos

### Ingrediente no declarado

Este programa es sintácticamente correcto, pero semánticamente inválido:

```text
Recipe "Prueba" {
    Add 200g harina;
    Mix harina azucar;
}
```

El error ocurre porque `azucar` se usa en `Mix`, pero nunca fue declarada con `Add`.

---

### Cantidad inválida

Este programa también es sintácticamente correcto, pero semánticamente inválido:

```text
Recipe "Prueba" {
    Add 0g harina;
}
```

El error ocurre porque la cantidad del ingrediente debe ser mayor que cero.

---

### Ingrediente repetido

```text
Recipe "Prueba" {
    Add 200g harina;
    Add 100g harina;
}
```

El error ocurre porque el ingrediente `harina` ya había sido declarado previamente.

---

## Traducción semántica elegida

La traducción elegida para **CookLang** será una estructura JSON que representa:

* el nombre de la receta;
* la lista de ingredientes;
* la lista de pasos;
* las estructuras condicionales;
* la cantidad de porciones.

Ejemplo en CookLang:

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

Traducción esperada a JSON:

```json
{
  "recipe": "Tarta de manzana",
  "ingredients": [
    {
      "name": "harina",
      "quantity": 200,
      "unit": "g"
    },
    {
      "name": "azucar",
      "quantity": 100,
      "unit": "g"
    },
    {
      "name": "manzanas",
      "quantity": 2,
      "unit": "pcs"
    }
  ],
  "steps": [
    {
      "action": "mix",
      "ingredients": ["harina", "azucar", "manzanas"]
    },
    {
      "condition": "tiene_horno",
      "then": {
        "action": "bake",
        "time": 45,
        "unit": "minutes"
      },
      "else": {
        "action": "chill",
        "time": 60,
        "unit": "minutes"
      }
    }
  ],
  "servings": 4
}
```

---

## Implementación de acciones semánticas con PLY

La implementación semántica se integra dentro de las reglas del parser de PLY.

Cada producción no solo reconoce una estructura sintáctica, sino que también devuelve un valor semántico.

Por ejemplo:

```python
def p_add_step(p):
    'add_step : ADD INT UNIT ID PYC'
    p[0] = {
        "type": "ingredient",
        "name": p[4],
        "quantity": p[2],
        "unit": p[3]
    }
```

La variable `p[0]` representa el resultado semántico de la producción.

---

### Código Python - Parser con acciones semánticas

```python
import json
import ply.yacc as yacc

# Se importan los tokens y el lexer desde el scanner
from scanner_cooklang import tokens, lexer


# ---------------------------------------
# Tabla de símbolos semántica
# ---------------------------------------

class SemanticContext:

    def __init__(self):
        self.ingredients = set()

    def add_ingredient(self, name):
        if name in self.ingredients:
            raise Exception(f"Error semántico: ingrediente repetido '{name}'")

        self.ingredients.add(name)

    def validate_ingredients(self, ingredients):
        for ingredient in ingredients:
            if ingredient not in self.ingredients:
                raise Exception(
                    f"Error semántico: ingrediente no declarado '{ingredient}'"
                )


context = SemanticContext()


# ---------------------------------------
# Programa principal
# ---------------------------------------

def p_program(p):
    'program : RECIPE STRING LLAVE_ABRE steps LLAVE_CIERRA'
    recipe_name = p[2][1:-1]

    ingredients = []
    recipe_steps = []
    servings = None

    for step in p[4]:
        if step.get("type") == "ingredient":
            ingredients.append({
                "name": step["name"],
                "quantity": step["quantity"],
                "unit": step["unit"]
            })

        elif step.get("type") == "servings":
            servings = step["value"]

        else:
            recipe_steps.append(step)

    p[0] = {
        "recipe": recipe_name,
        "ingredients": ingredients,
        "steps": recipe_steps,
        "servings": servings
    }


# ---------------------------------------
# Lista de instrucciones
# ---------------------------------------

def p_steps_recursive(p):
    'steps : step steps'
    p[0] = [p[1]] + p[2]


def p_steps_single(p):
    'steps : step'
    p[0] = [p[1]]


def p_step(p):
    '''
    step : add_step
         | mix_step
         | bake_step
         | chill_step
         | conditional_step
         | serve_step
    '''
    p[0] = p[1]


# ---------------------------------------
# Instrucción Add
# ---------------------------------------

def p_add_step(p):
    'add_step : ADD INT UNIT ID PYC'

    if p[2] <= 0:
        raise Exception("Error semántico: la cantidad debe ser mayor que cero")

    context.add_ingredient(p[4])

    p[0] = {
        "type": "ingredient",
        "name": p[4],
        "quantity": p[2],
        "unit": p[3]
    }


# ---------------------------------------
# Instrucción Mix
# ---------------------------------------

def p_mix_step(p):
    'mix_step : MIX id_list PYC'

    context.validate_ingredients(p[2])

    p[0] = {
        "action": "mix",
        "ingredients": p[2]
    }


# ---------------------------------------
# Instrucción Bake
# ---------------------------------------

def p_bake_step(p):
    'bake_step : BAKE INT MINUTES PYC'

    if p[2] <= 0:
        raise Exception("Error semántico: el tiempo de cocción debe ser mayor que cero")

    p[0] = {
        "action": "bake",
        "time": p[2],
        "unit": "minutes"
    }


# ---------------------------------------
# Instrucción Chill
# ---------------------------------------

def p_chill_step(p):
    'chill_step : CHILL INT MINUTES PYC'

    if p[2] <= 0:
        raise Exception("Error semántico: el tiempo de enfriado debe ser mayor que cero")

    p[0] = {
        "action": "chill",
        "time": p[2],
        "unit": "minutes"
    }


# ---------------------------------------
# Instrucción Serve
# ---------------------------------------

def p_serve_step(p):
    'serve_step : SERVE INT PORTIONS PYC'

    if p[2] <= 0:
        raise Exception("Error semántico: las porciones deben ser mayores que cero")

    p[0] = {
        "type": "servings",
        "value": p[2]
    }


# ---------------------------------------
# Condicional
# ---------------------------------------

def p_conditional_step(p):
    'conditional_step : IF condition THEN action ELSE action PYC'

    p[0] = {
        "condition": p[2],
        "then": p[4],
        "else": p[6]
    }


def p_condition(p):
    'condition : ID'
    p[0] = p[1]


# ---------------------------------------
# Acciones dentro del condicional
# ---------------------------------------

def p_action_bake(p):
    'action : BAKE INT MINUTES'

    if p[2] <= 0:
        raise Exception("Error semántico: el tiempo de cocción debe ser mayor que cero")

    p[0] = {
        "action": "bake",
        "time": p[2],
        "unit": "minutes"
    }


def p_action_chill(p):
    'action : CHILL INT MINUTES'

    if p[2] <= 0:
        raise Exception("Error semántico: el tiempo de enfriado debe ser mayor que cero")

    p[0] = {
        "action": "chill",
        "time": p[2],
        "unit": "minutes"
    }


def p_action_mix(p):
    'action : MIX id_list'

    context.validate_ingredients(p[2])

    p[0] = {
        "action": "mix",
        "ingredients": p[2]
    }


# ---------------------------------------
# Lista de identificadores
# ---------------------------------------

def p_id_list_recursive(p):
    'id_list : ID id_list'
    p[0] = [p[1]] + p[2]


def p_id_list_single(p):
    'id_list : ID'
    p[0] = [p[1]]


# ---------------------------------------
# Manejo de errores sintácticos
# ---------------------------------------

def p_error(p):
    if p:
        raise SyntaxError(
            f"Error sintáctico en token {p.type} con valor {p.value}"
        )
    else:
        raise SyntaxError("Error sintáctico: fin de entrada inesperado")


# ---------------------------------------
# Construcción del parser
# ---------------------------------------

parser = yacc.yacc()


# ---------------------------------------
# Función de traducción
# ---------------------------------------

def traducir_a_json(codigo):
    global context

    context = SemanticContext()

    resultado = parser.parse(codigo)

    return json.dumps(resultado, indent=2, ensure_ascii=False)
```

---

## Prueba con programa válido

```python
programa = '''
Recipe "Tarta de manzana" {
    Add 200g harina;
    Add 100g azucar;
    Add 2pcs manzanas;
    Mix harina azucar manzanas;
    If tiene_horno then Bake 45 minutes else Chill 60 minutes;
    Serve 4 portions;
}
'''

print(traducir_a_json(programa))
```

### Salida esperada

```json
{
  "recipe": "Tarta de manzana",
  "ingredients": [
    {
      "name": "harina",
      "quantity": 200,
      "unit": "g"
    },
    {
      "name": "azucar",
      "quantity": 100,
      "unit": "g"
    },
    {
      "name": "manzanas",
      "quantity": 2,
      "unit": "pcs"
    }
  ],
  "steps": [
    {
      "action": "mix",
      "ingredients": [
        "harina",
        "azucar",
        "manzanas"
      ]
    },
    {
      "condition": "tiene_horno",
      "then": {
        "action": "bake",
        "time": 45,
        "unit": "minutes"
      },
      "else": {
        "action": "chill",
        "time": 60,
        "unit": "minutes"
      }
    }
  ],
  "servings": 4
}
```

---

## Prueba con ingrediente no declarado

```python
programa = '''
Recipe "Prueba" {
    Add 200g harina;
    Mix harina azucar;
}
'''

print(traducir_a_json(programa))
```

### Salida esperada

```text
Error semántico: ingrediente no declarado 'azucar'
```

---

## Prueba con cantidad inválida

```python
programa = '''
Recipe "Prueba" {
    Add 0g harina;
}
'''

print(traducir_a_json(programa))
```

### Salida esperada

```text
Error semántico: la cantidad debe ser mayor que cero
```

---

## Prueba con ingrediente repetido

```python
programa = '''
Recipe "Prueba" {
    Add 200g harina;
    Add 100g harina;
}
'''

print(traducir_a_json(programa))
```

### Salida esperada

```text
Error semántico: ingrediente repetido 'harina'
```

---

## Conclusión

Las acciones semánticas permiten verificar que un programa válido sintácticamente también tenga sentido dentro del dominio de las recetas de cocina.

En CookLang, estas acciones permiten validar cantidades, tiempos, porciones e ingredientes declarados.

Además, permiten traducir el programa fuente a una estructura JSON, que representa la receta de forma ordenada y reutilizable por otros sistemas.
