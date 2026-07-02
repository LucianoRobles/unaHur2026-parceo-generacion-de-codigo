# Scanner de CookLang

## LEXEMAS SCANNER - CookLang

| Token | Lexema |
|---|---|
| `RECIPE` | `Recipe` |
| `ADD` | `Add` |
| `MIX` | `Mix` |
| `BAKE` | `Bake` |
| `CHILL` | `Chill` |
| `IF` | `If` |
| `THEN` | `then` |
| `ELSE` | `else` |
| `SERVE` | `Serve` |
| `MINUTES` | `minutes` |
| `PORTIONS` | `portions` |
| `ID` | Letra seguida de letras, dígitos o `_` |
| `INT` | Uno o más dígitos |
| `STRING` | Texto encerrado entre comillas dobles |
| `UNIT` | `g`, `ml` o `pcs` |
| `LLAVE_ABRE` | `{` |
| `LLAVE_CIERRA` | `}` |
| `PYC` | `;` |
| `EOF` | Fin de archivo |

## Aclaración sobre identificadores

Un identificador puede representar nombres de ingredientes o condiciones.

Ejemplos válidos:

```text
harina
azucar
manzanas
tiene_horno
masa_liquida
```

Ejemplos inválidos:

```text
_ingrediente
masa_
masa__liquida
```

Regla léxica:

```text
ID → letra (letra | digito | "_")*
```

Validaciones adicionales:

- No puede comenzar con `_`.
- No puede terminar con `_`.
- No puede contener `__`.

---

# Tabla de transiciones del SCANNER

| Q | L | D | `_` | `"` | `{` | `}` | `;` | Blanco | EOF | OTRO | TOKEN | RETROCESO |
|---|---|---|---|---|---|---|---|---|---|---|---|---:|
| `>q0` | q1 | q4 | ERROR | q6 | q9 | q10 | q11 | q0 | FIN | ERROR | - | - |
| q1 | q1 | q1 | q1 | q2 | q2 | q2 | q2 | q2 | q3 | q2 | - | - |
| `*q2` | - | - | - | - | - | - | - | - | - | - | ID / RESERVADA / UNIT | 1 |
| `*q3` | - | - | - | - | - | - | - | - | - | - | ID / RESERVADA / UNIT | 0 |
| q4 | q5 | q4 | q5 | q5 | q5 | q5 | q5 | q5 | q12 | q5 | - | - |
| `*q5` | - | - | - | - | - | - | - | - | - | - | INT | 1 |
| q6 | q6 | q6 | q6 | q7 | q6 | q6 | q6 | q6 | ERROR | q6 | - | - |
| `*q7` | - | - | - | - | - | - | - | - | - | - | STRING | 0 |
| `*q9` | - | - | - | - | - | - | - | - | - | - | LLAVE_ABRE | 0 |
| `*q10` | - | - | - | - | - | - | - | - | - | - | LLAVE_CIERRA | 0 |
| `*q11` | - | - | - | - | - | - | - | - | - | - | PYC | 0 |
| `*q12` | - | - | - | - | - | - | - | - | - | - | INT | 0 |

## Explicación breve de la tabla

- `q0` es el estado inicial.
- Si lee una letra, pasa a `q1` para formar identificadores, palabras reservadas o unidades.
- Si lee un dígito, pasa a `q4` para formar números enteros.
- Si lee `"`, pasa a `q6` para formar un `STRING`.
- Si lee `{`, `}` o `;`, reconoce directamente el token correspondiente.
- Los espacios, tabulaciones y saltos de línea se ignoran.
- Cuando el autómata reconoce una cadena alfabética, primero la acepta como posible `ID`. Luego se consulta una tabla de palabras reservadas y unidades para determinar el token final.
- El retroceso sirve para devolver el último carácter leído cuando ese carácter no pertenece al token actual.

Ejemplos de clasificación posterior:

```text
Recipe  → RECIPE
Add     → ADD
g       → UNIT
harina  → ID
```

---

# CÓDIGO PYTHON DEL SCANNER CON PLY

Para la implementación práctica del scanner se utiliza la herramienta **PLY**, específicamente el módulo `ply.lex`.

La tabla de transiciones anterior representa la definición formal del autómata.  
En cambio, PLY permite implementar el scanner mediante expresiones regulares y funciones asociadas a cada token.

```python
import ply.lex as lex


# ---------------------------------------
# Palabras reservadas y unidades
# ---------------------------------------

reserved = {
    "Recipe": "RECIPE",
    "Add": "ADD",
    "Mix": "MIX",
    "Bake": "BAKE",
    "Chill": "CHILL",
    "If": "IF",
    "then": "THEN",
    "else": "ELSE",
    "Serve": "SERVE",
    "minutes": "MINUTES",
    "portions": "PORTIONS",
    "g": "UNIT",
    "ml": "UNIT",
    "pcs": "UNIT"
}


# ---------------------------------------
# Lista de tokens
# ---------------------------------------

tokens = (
    "RECIPE",
    "ADD",
    "MIX",
    "BAKE",
    "CHILL",
    "IF",
    "THEN",
    "ELSE",
    "SERVE",
    "MINUTES",
    "PORTIONS",
    "ID",
    "INT",
    "STRING",
    "UNIT",
    "LLAVE_ABRE",
    "LLAVE_CIERRA",
    "PYC"
)


# ---------------------------------------
# Expresiones regulares simples
# ---------------------------------------

t_LLAVE_ABRE = r"\{"
t_LLAVE_CIERRA = r"\}"
t_PYC = r";"


# ---------------------------------------
# Caracteres ignorados
# ---------------------------------------

t_ignore = " \t\r"


# ---------------------------------------
# Token STRING
# Reconoce texto encerrado entre comillas dobles
# ---------------------------------------

def t_STRING(t):
    r'"[^"\n]*"'
    return t


# ---------------------------------------
# Token INT
# Reconoce uno o más dígitos
# ---------------------------------------

def t_INT(t):
    r"\d+"
    t.value = int(t.value)
    return t


# ---------------------------------------
# Token ID, palabras reservadas y unidades
# ---------------------------------------

def t_ID(t):
    r"[a-zA-Z][a-zA-Z0-9_]*"

    if t.value.endswith("_"):
        raise SyntaxError(f"Identificador inválido: {t.value}")

    if "__" in t.value:
        raise SyntaxError(f"Identificador inválido: {t.value}")

    t.type = reserved.get(t.value, "ID")
    return t


# ---------------------------------------
# Contador de líneas
# ---------------------------------------

def t_newline(t):
    r"\n+"
    t.lexer.lineno += len(t.value)


# ---------------------------------------
# Manejo de errores léxicos
# ---------------------------------------

def t_error(t):
    raise SyntaxError(
        f"Caracter no reconocido '{t.value[0]}' en la línea {t.lexer.lineno}"
    )


# ---------------------------------------
# Construcción del lexer
# ---------------------------------------

lexer = lex.lex()


# ---------------------------------------
# Función auxiliar para probar el scanner
# ---------------------------------------

def analizar_codigo(codigo):
    lexer.input(codigo)

    tokens_encontrados = []

    while True:
        tok = lexer.token()

        if not tok:
            break

        tokens_encontrados.append((tok.type, tok.value))

    return tokens_encontrados
```

---

# Prueba simple del scanner

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

resultado = analizar_codigo(programa)

for token, lexema in resultado:
    print(token, "->", lexema)
```

## Salida esperada

```text
RECIPE -> Recipe
STRING -> "Tarta de manzana"
LLAVE_ABRE -> {
ADD -> Add
INT -> 200
UNIT -> g
ID -> harina
PYC -> ;
ADD -> Add
INT -> 100
UNIT -> g
ID -> azucar
PYC -> ;
ADD -> Add
INT -> 2
UNIT -> pcs
ID -> manzanas
PYC -> ;
MIX -> Mix
ID -> harina
ID -> azucar
ID -> manzanas
PYC -> ;
IF -> If
ID -> tiene_horno
THEN -> then
BAKE -> Bake
INT -> 45
MINUTES -> minutes
ELSE -> else
CHILL -> Chill
INT -> 60
MINUTES -> minutes
PYC -> ;
SERVE -> Serve
INT -> 4
PORTIONS -> portions
PYC -> ;
LLAVE_CIERRA -> }
```

---

# Prueba con identificador inválido

```python
programa = '''
Recipe "Prueba" {
    Add 200g masa_;
}
'''

resultado = analizar_codigo(programa)
```

## Salida esperada

```text
SyntaxError: Identificador inválido: masa_
```

---

# Prueba con cadena sin cerrar

```python
programa = '''
Recipe "Tarta de manzana {
    Add 200g harina;
}
'''

resultado = analizar_codigo(programa)
```

## Salida esperada

```text
SyntaxError: Caracter no reconocido '"' en la línea 2
```

---

# Nota importante

En CookLang, una cantidad como:

```text
200g
```

se reconoce como dos tokens separados:

```text
INT  -> 200
UNIT -> g
```

Lo mismo ocurre con:

```text
200ml
2pcs
```

Esto sucede porque el scanner reconoce primero el número entero y luego reconoce la unidad como una palabra reservada clasificada como `UNIT`.

Esto simplifica el parser, porque la regla sintáctica queda:

```text
quantity → INT UNIT
```

Por lo tanto, el lenguaje acepta tanto:

```text
Add 200g harina;
```

como:

```text
Add 200 g harina;
```

---

# Conclusión

El scanner de CookLang fue definido formalmente mediante una tabla de lexemas y una tabla de transiciones.

Para la implementación práctica en Python se utilizó **PLY**, que permite definir los tokens mediante expresiones regulares.

Esta implementación reconoce correctamente palabras reservadas, identificadores, números enteros, cadenas de texto, unidades de medida y símbolos especiales, dejando preparada la secuencia de tokens para ser utilizada por el parser.
