# Parser de CookLang

## Gramática Independiente del Contexto

La gramática de **CookLang** se define como una Gramática Independiente del Contexto estricta.

El lenguaje no acepta el programa vacío, por lo tanto no se utiliza λ en el axioma.

---

## Terminales

```text
RECIPE
STRING
LLAVE_ABRE
LLAVE_CIERRA
ADD
MIX
BAKE
CHILL
IF
THEN
ELSE
SERVE
INT
UNIT
ID
MINUTES
PORTIONS
PYC
EOF
```

---

## No terminales

```text
P
STEPS
STEP
ADD_STEP
MIX_STEP
BAKE_STEP
CHILL_STEP
CONDITIONAL_STEP
SERVE_STEP
ACTION
CONDITION
IDLIST
```

---

## Axioma

```text
P
```

---

## Producciones

```text
P → RECIPE STRING LLAVE_ABRE STEPS LLAVE_CIERRA EOF

STEPS → STEP STEPS
STEPS → STEP

STEP → ADD_STEP
STEP → MIX_STEP
STEP → BAKE_STEP
STEP → CHILL_STEP
STEP → CONDITIONAL_STEP
STEP → SERVE_STEP

ADD_STEP → ADD INT UNIT ID PYC

MIX_STEP → MIX IDLIST PYC

BAKE_STEP → BAKE INT MINUTES PYC

CHILL_STEP → CHILL INT MINUTES PYC

CONDITIONAL_STEP → IF CONDITION THEN ACTION ELSE ACTION PYC

SERVE_STEP → SERVE INT PORTIONS PYC

ACTION → BAKE INT MINUTES
ACTION → CHILL INT MINUTES
ACTION → MIX IDLIST

CONDITION → ID

IDLIST → ID IDLIST
IDLIST → ID
```

---

## Tabla de parseo del ASDB

| Estado | Transición |
| ------ | ---------- |
| q0 | δ(q0,λ,λ) = {(q1,*)} |
| q1 | δ(q1,λ,λ) = {(q2,P)} |
| q2 | δ(q2,λ,P) = {(q2,RECIPE STRING LLAVE_ABRE STEPS LLAVE_CIERRA EOF)} |
| q2 | δ(q2,λ,STEPS) = {(q2,STEP STEPS)} |
| q2 | δ(q2,λ,STEPS) = {(q2,STEP)} |
| q2 | δ(q2,λ,STEP) = {(q2,ADD_STEP)} |
| q2 | δ(q2,λ,STEP) = {(q2,MIX_STEP)} |
| q2 | δ(q2,λ,STEP) = {(q2,BAKE_STEP)} |
| q2 | δ(q2,λ,STEP) = {(q2,CHILL_STEP)} |
| q2 | δ(q2,λ,STEP) = {(q2,CONDITIONAL_STEP)} |
| q2 | δ(q2,λ,STEP) = {(q2,SERVE_STEP)} |
| q2 | δ(q2,λ,ADD_STEP) = {(q2,ADD INT UNIT ID PYC)} |
| q2 | δ(q2,λ,MIX_STEP) = {(q2,MIX IDLIST PYC)} |
| q2 | δ(q2,λ,BAKE_STEP) = {(q2,BAKE INT MINUTES PYC)} |
| q2 | δ(q2,λ,CHILL_STEP) = {(q2,CHILL INT MINUTES PYC)} |
| q2 | δ(q2,λ,CONDITIONAL_STEP) = {(q2,IF CONDITION THEN ACTION ELSE ACTION PYC)} |
| q2 | δ(q2,λ,SERVE_STEP) = {(q2,SERVE INT PORTIONS PYC)} |
| q2 | δ(q2,λ,ACTION) = {(q2,BAKE INT MINUTES)} |
| q2 | δ(q2,λ,ACTION) = {(q2,CHILL INT MINUTES)} |
| q2 | δ(q2,λ,ACTION) = {(q2,MIX IDLIST)} |
| q2 | δ(q2,λ,CONDITION) = {(q2,ID)} |
| q2 | δ(q2,λ,IDLIST) = {(q2,ID IDLIST)} |
| q2 | δ(q2,λ,IDLIST) = {(q2,ID)} |
| q2 | δ(q2,RECIPE,RECIPE) = {(q2,λ)} |
| q2 | δ(q2,STRING,STRING) = {(q2,λ)} |
| q2 | δ(q2,LLAVE_ABRE,LLAVE_ABRE) = {(q2,λ)} |
| q2 | δ(q2,LLAVE_CIERRA,LLAVE_CIERRA) = {(q2,λ)} |
| q2 | δ(q2,ADD,ADD) = {(q2,λ)} |
| q2 | δ(q2,MIX,MIX) = {(q2,λ)} |
| q2 | δ(q2,BAKE,BAKE) = {(q2,λ)} |
| q2 | δ(q2,CHILL,CHILL) = {(q2,λ)} |
| q2 | δ(q2,IF,IF) = {(q2,λ)} |
| q2 | δ(q2,THEN,THEN) = {(q2,λ)} |
| q2 | δ(q2,ELSE,ELSE) = {(q2,λ)} |
| q2 | δ(q2,SERVE,SERVE) = {(q2,λ)} |
| q2 | δ(q2,INT,INT) = {(q2,λ)} |
| q2 | δ(q2,UNIT,UNIT) = {(q2,λ)} |
| q2 | δ(q2,ID,ID) = {(q2,λ)} |
| q2 | δ(q2,MINUTES,MINUTES) = {(q2,λ)} |
| q2 | δ(q2,PORTIONS,PORTIONS) = {(q2,λ)} |
| q2 | δ(q2,PYC,PYC) = {(q2,λ)} |
| q2 | δ(q2,EOF,EOF) = {(q2,λ)} |
| q2 | δ(q2,λ,*) = {(q3,λ)} |
| q3 | Accept |

---

## Parser con PLY

Para la implementación práctica del parser se utiliza **PLY**, específicamente el módulo `ply.yacc`.

El scanner ya definido con `ply.lex` genera los tokens, y el parser consume esa secuencia para validar que el programa respete la gramática de CookLang.

---

### Código Python del parser con PLY

```python
import ply.yacc as yacc

# Se importan los tokens y el lexer definidos en el scanner.
# Si el scanner está en el mismo archivo, no hace falta importar.
# Si está en otro archivo, por ejemplo scanner_cooklang.py, se usaría:
# from scanner_cooklang import tokens, lexer


# ---------------------------------------
# Producción inicial
# ---------------------------------------

def p_programa(p):
    '''programa : RECIPE STRING LLAVE_ABRE steps LLAVE_CIERRA'''
    p[0] = {
        "type": "recipe",
        "name": p[2],
        "steps": p[4]
    }


# ---------------------------------------
# Lista de instrucciones
# ---------------------------------------

def p_steps_recursivo(p):
    '''steps : step steps'''
    p[0] = [p[1]] + p[2]


def p_steps_base(p):
    '''steps : step'''
    p[0] = [p[1]]


# ---------------------------------------
# Instrucciones posibles
# ---------------------------------------

def p_step(p):
    '''step : add_step
            | mix_step
            | bake_step
            | chill_step
            | conditional_step
            | serve_step'''
    p[0] = p[1]


# ---------------------------------------
# Instrucción Add
# ---------------------------------------

def p_add_step(p):
    '''add_step : ADD INT UNIT ID PYC'''
    p[0] = {
        "type": "add",
        "quantity": p[2],
        "unit": p[3],
        "ingredient": p[4]
    }


# ---------------------------------------
# Instrucción Mix
# ---------------------------------------

def p_mix_step(p):
    '''mix_step : MIX idlist PYC'''
    p[0] = {
        "type": "mix",
        "ingredients": p[2]
    }


# ---------------------------------------
# Instrucción Bake
# ---------------------------------------

def p_bake_step(p):
    '''bake_step : BAKE INT MINUTES PYC'''
    p[0] = {
        "type": "bake",
        "time": p[2],
        "unit": "minutes"
    }


# ---------------------------------------
# Instrucción Chill
# ---------------------------------------

def p_chill_step(p):
    '''chill_step : CHILL INT MINUTES PYC'''
    p[0] = {
        "type": "chill",
        "time": p[2],
        "unit": "minutes"
    }


# ---------------------------------------
# Instrucción Serve
# ---------------------------------------

def p_serve_step(p):
    '''serve_step : SERVE INT PORTIONS PYC'''
    p[0] = {
        "type": "serve",
        "portions": p[2]
    }


# ---------------------------------------
# Condicional If then else
# ---------------------------------------

def p_conditional_step(p):
    '''conditional_step : IF condition THEN action ELSE action PYC'''
    p[0] = {
        "type": "if",
        "condition": p[2],
        "then": p[4],
        "else": p[6]
    }


# ---------------------------------------
# Acciones permitidas dentro del condicional
# ---------------------------------------

def p_action_bake(p):
    '''action : BAKE INT MINUTES'''
    p[0] = {
        "type": "bake",
        "time": p[2],
        "unit": "minutes"
    }


def p_action_chill(p):
    '''action : CHILL INT MINUTES'''
    p[0] = {
        "type": "chill",
        "time": p[2],
        "unit": "minutes"
    }


def p_action_mix(p):
    '''action : MIX idlist'''
    p[0] = {
        "type": "mix",
        "ingredients": p[2]
    }


# ---------------------------------------
# Condición
# ---------------------------------------

def p_condition(p):
    '''condition : ID'''
    p[0] = p[1]


# ---------------------------------------
# Lista de identificadores
# ---------------------------------------

def p_idlist_recursivo(p):
    '''idlist : ID idlist'''
    p[0] = [p[1]] + p[2]


def p_idlist_base(p):
    '''idlist : ID'''
    p[0] = [p[1]]


# ---------------------------------------
# Manejo de errores sintácticos
# ---------------------------------------

def p_error(p):
    if p:
        raise SyntaxError(
            f"Error sintáctico cerca de '{p.value}' en la línea {p.lineno}"
        )
    else:
        raise SyntaxError("Error sintáctico: fin inesperado de entrada")


# ---------------------------------------
# Construcción del parser
# ---------------------------------------

parser = yacc.yacc(start="programa")
```

---

## Prueba del parser

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

resultado = parser.parse(programa, lexer=lexer)
print(resultado)
```

### Salida esperada

```text
{
    'type': 'recipe',
    'name': '"Tarta de manzana"',
    'steps': [
        {'type': 'add', 'quantity': 200, 'unit': 'g', 'ingredient': 'harina'},
        {'type': 'add', 'quantity': 100, 'unit': 'g', 'ingredient': 'azucar'},
        {'type': 'add', 'quantity': 2, 'unit': 'pcs', 'ingredient': 'manzanas'},
        {'type': 'mix', 'ingredients': ['harina', 'azucar', 'manzanas']},
        {
            'type': 'if',
            'condition': 'tiene_horno',
            'then': {'type': 'bake', 'time': 45, 'unit': 'minutes'},
            'else': {'type': 'chill', 'time': 60, 'unit': 'minutes'}
        },
        {'type': 'serve', 'portions': 4}
    ]
}
```

---

## Prueba con `Mix` dentro del condicional

```python
programa = '''
Recipe "Pan rapido" {
    Add 300g harina;
    Add 200ml agua;
    Add 10g sal;
    Mix harina agua sal;
    If masa_liquida then Mix harina agua else Bake 30 minutes;
    Serve 2 portions;
}
'''

resultado = parser.parse(programa, lexer=lexer)
print(resultado)
```

### Salida esperada

```text
{
    'type': 'recipe',
    'name': '"Pan rapido"',
    'steps': [
        {'type': 'add', 'quantity': 300, 'unit': 'g', 'ingredient': 'harina'},
        {'type': 'add', 'quantity': 200, 'unit': 'ml', 'ingredient': 'agua'},
        {'type': 'add', 'quantity': 10, 'unit': 'g', 'ingredient': 'sal'},
        {'type': 'mix', 'ingredients': ['harina', 'agua', 'sal']},
        {
            'type': 'if',
            'condition': 'masa_liquida',
            'then': {'type': 'mix', 'ingredients': ['harina', 'agua']},
            'else': {'type': 'bake', 'time': 30, 'unit': 'minutes'}
        },
        {'type': 'serve', 'portions': 2}
    ]
}
```

---

## Prueba con programa inválido

```python
programa = '''
Recipe "Tarta de manzana" {
    Add harina 200g;
}
'''

resultado = parser.parse(programa, lexer=lexer)
```

### Salida esperada

```text
SyntaxError: Error sintáctico cerca de 'harina'
```

---

## Relación entre scanner y parser

El scanner transforma el texto fuente en tokens.

Por ejemplo:

```text
Add 200g harina;
```

se transforma en:

```text
ADD  -> Add
INT  -> 200
UNIT -> g
ID   -> harina
PYC  -> ;
```

Luego el parser verifica que esos tokens respeten la producción:

```text
ADD_STEP → ADD INT UNIT ID PYC
```

Si la secuencia coincide con la producción, la instrucción es sintácticamente válida.

---

## Validaciones que todavía no realiza el parser

El parser solamente valida la estructura sintáctica del programa.

Todavía no verifica cuestiones semánticas como:

- Si un ingrediente usado en `Mix` fue declarado previamente con `Add`.
- Si la cantidad de un ingrediente es mayor que cero.
- Si el tiempo de cocción o enfriado es mayor que cero.
- Si la cantidad de porciones es mayor que cero.
- Si hay ingredientes repetidos.
- Si una condición tiene valor verdadero o falso.

Estas validaciones corresponden a la etapa de acciones semánticas.

---

## Conclusión

El parser de CookLang fue definido formalmente mediante una Gramática Independiente del Contexto y su correspondiente tabla de parseo de un Autómata a Pila (ASDB).

Para la implementación práctica en Python se utilizó **PLY**, específicamente el módulo `ply.yacc`, que construye el parser a partir de las producciones expresadas como docstrings de funciones.

Esta implementación valida que la secuencia de tokens generada por el scanner respete la estructura sintáctica del lenguaje, dejando el árbol de la receta preparado para que la etapa de acciones semánticas verifique su significado y lo traduzca a JSON.
