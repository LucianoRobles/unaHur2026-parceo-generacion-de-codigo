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
- Si lee una letra, pasa a `q1` para formar identificadores o palabras reservadas.
- Si lee un dígito, pasa a `q4` para formar números enteros.
- Si lee `"`, pasa a `q6` para formar un `STRING`.
- Si lee `{`, `}` o `;`, reconoce directamente el token correspondiente.
- Los espacios, tabulaciones y saltos de línea se ignoran.
- Cuando se reconoce un `ID`, luego se verifica si en realidad es una palabra reservada o una unidad de medida.
- El retroceso sirve para devolver el último carácter leído cuando ese carácter no pertenece al token actual.

---

# CÓDIGO PYTHON DEL SCANNER

```python
class LexerCookLang:

    ERROR = -1
    ACEPTAR = 0
    FIN = 1

    def __init__(self, texto):
        self.texto = texto
        self.cursor = -1
        self.inicio_lexema = 0
        self.token = None
        self.lexema = None
        self.retroceso = None

        self.palabras_reservadas = {
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
            "portions": "PORTIONS"
        }

        self.unidades = {
            "g": "UNIT",
            "ml": "UNIT",
            "pcs": "UNIT"
        }

    def getCaracter(self):
        if (self.cursor + 1) < len(self.texto):
            self.cursor += 1
            return self.texto[self.cursor]
        return "EOF"

    def esLetra(self, c):
        return len(c) == 1 and c.isalpha()

    def esDigito(self, c):
        return len(c) == 1 and c >= "0" and c <= "9"

    def esBlanco(self, c):
        return c == " " or c == "\t" or c == "\n" or c == "\r"

    def esCaracterID(self, c):
        return self.esLetra(c) or self.esDigito(c) or c == "_"

    def validarID(self, lexema):
        if len(lexema) == 0:
            return False

        if not self.esLetra(lexema[0]):
            return False

        if lexema[-1] == "_":
            return False

        if "__" in lexema:
            return False

        return True

    def clasificarID(self, lexema):
        if lexema in self.palabras_reservadas:
            return self.palabras_reservadas[lexema]

        if lexema in self.unidades:
            return self.unidades[lexema]

        if self.validarID(lexema):
            return "ID"

        raise Exception(f"Identificador inválido: {lexema}")

    def siguienteToken(self):
        estado = 0
        self.token = None
        self.lexema = None
        self.retroceso = 0

        while True:
            c = self.getCaracter()

            if estado == 0:
                self.inicio_lexema = self.cursor

                if c == "EOF":
                    self.token = "EOF"
                    self.lexema = "EOF"
                    return self.token, self.lexema

                elif self.esBlanco(c):
                    estado = 0

                elif self.esLetra(c):
                    estado = 1

                elif self.esDigito(c):
                    estado = 4

                elif c == '"':
                    estado = 6

                elif c == "{":
                    self.token = "LLAVE_ABRE"
                    self.lexema = c
                    return self.token, self.lexema

                elif c == "}":
                    self.token = "LLAVE_CIERRA"
                    self.lexema = c
                    return self.token, self.lexema

                elif c == ";":
                    self.token = "PYC"
                    self.lexema = c
                    return self.token, self.lexema

                else:
                    raise Exception(f"Caracter no reconocido: {c}")

            elif estado == 1:
                if self.esCaracterID(c):
                    estado = 1
                elif c == "EOF":
                    self.lexema = self.texto[self.inicio_lexema:self.cursor + 1]
                    self.token = self.clasificarID(self.lexema)
                    return self.token, self.lexema
                else:
                    self.cursor -= 1
                    self.lexema = self.texto[self.inicio_lexema:self.cursor + 1]
                    self.token = self.clasificarID(self.lexema)
                    return self.token, self.lexema

            elif estado == 4:
                if self.esDigito(c):
                    estado = 4
                elif c == "EOF":
                    self.lexema = self.texto[self.inicio_lexema:self.cursor + 1]
                    self.token = "INT"
                    return self.token, self.lexema
                else:
                    self.cursor -= 1
                    self.lexema = self.texto[self.inicio_lexema:self.cursor + 1]
                    self.token = "INT"
                    return self.token, self.lexema

            elif estado == 6:
                if c == "EOF":
                    raise Exception("Cadena sin cerrar")

                elif c == '"':
                    self.lexema = self.texto[self.inicio_lexema:self.cursor + 1]
                    self.token = "STRING"
                    return self.token, self.lexema

                else:
                    estado = 6
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

lexer = LexerCookLang(programa)

while True:
    token, lexema = lexer.siguienteToken()
    print(token, "->", lexema)

    if token == "EOF":
        break
```

## Salida esperada aproximada

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
EOF -> EOF
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

Esto simplifica el parser, porque la regla sintáctica queda:

```text
quantity → INT UNIT
```
