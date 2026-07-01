# Lenguaje a crear: CookLang

## Objetivo

El objetivo de **CookLang** es definir un lenguaje de programación simple orientado a la descripción de recetas de cocina.

El lenguaje permite representar una receta mediante instrucciones estructuradas, como agregar ingredientes, mezclar, cocinar, enfriar, indicar cantidad de porciones y ejecutar pasos condicionales según una condición determinada.

La idea principal es que una receta escrita en CookLang pueda luego ser procesada por un scanner, un parser y acciones semánticas para generar una representación estructurada de la receta, por ejemplo en formato JSON o como una lista ordenada de pasos ejecutables por una aplicación de cocina.

Ejemplo general:

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

---

## Alcance

El lenguaje **CookLang** permite:

* Definir una receta con un nombre.
* Agregar ingredientes con cantidades y unidades.
* Mezclar ingredientes.
* Cocinar una preparación durante una cantidad determinada de minutos.
* Enfriar una preparación durante una cantidad determinada de minutos.
* Utilizar una estructura condicional simple para decidir entre dos acciones posibles.
* Indicar la cantidad de porciones que rinde la receta.

El lenguaje no contempla, en esta primera versión:

* Cálculo nutricional.
* Temperaturas de cocción.
* Conversión automática de unidades.
* Bucles o repeticiones.
* Validación de disponibilidad real de ingredientes.
* Subrecetas o funciones.
* Operaciones matemáticas sobre cantidades.

El alcance está limitado a recetas simples, escritas con una estructura clara y fácil de analizar mediante herramientas de análisis léxico, sintáctico y semántico.

---

## Especificaciones léxicas

El lenguaje está compuesto por palabras reservadas, identificadores, cadenas de texto, números, unidades de medida y símbolos especiales.

### Palabras reservadas

```text
Recipe
Add
Mix
Bake
Chill
If
then
else
Serve
minutes
portions
```

### Identificadores

Los identificadores representan nombres de ingredientes o condiciones.

Ejemplos:

```text
harina
azucar
manzanas
tiene_horno
masa_liquida
```

Regla léxica:

```text
ID → letra (letra | digito | "_")*
```

### Números enteros

Los números enteros representan cantidades, tiempos o cantidad de porciones.

Ejemplos:

```text
200
100
2
45
60
4
```

Regla léxica:

```text
INT → digito+
```

### Cadenas de texto

Las cadenas de texto se utilizan para definir el nombre de la receta.

Ejemplo:

```text
"Tarta de manzana"
```

Regla léxica:

```text
STRING → "\"" caracter* "\""
```

### Unidades de medida

Las unidades permitidas son:

```text
g
ml
pcs
```

Donde:

* `g` representa gramos.
* `ml` representa mililitros.
* `pcs` representa unidades o piezas.

### Símbolos especiales

```text
{
}
;
```

Se utilizan para delimitar bloques e instrucciones.

---

## Especificaciones sintácticas

La estructura general de una receta en **CookLang** comienza con la palabra reservada `Recipe`, seguida del nombre de la receta entre comillas, y luego un bloque de instrucciones encerrado entre llaves.

Gramática propuesta:

```text
recipe → Recipe STRING { step+ }

step → addStep
     | mixStep
     | bakeStep
     | chillStep
     | conditionalStep
     | serveStep

addStep → Add quantity ID ;

mixStep → Mix ID+ ;

bakeStep → Bake INT minutes ;

chillStep → Chill INT minutes ;

conditionalStep → If condition then action else action ;

serveStep → Serve INT portions ;

action → Bake INT minutes
       | Chill INT minutes
       | Mix ID+

condition → ID

quantity → INT unit

unit → g
     | ml
     | pcs
```

Ejemplo válido:

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

Ejemplo válido usando `Mix` dentro del condicional:

```text
Recipe "Pan rapido" {
    Add 300g harina;
    Add 200ml agua;
    Add 10g sal;
    Mix harina agua sal;
    If masa_liquida then Mix harina masa else Bake 30 minutes;
    Serve 2 portions;
}
```

Ejemplo inválido:

```text
Recipe "Tarta de manzana" {
    Add harina 200g;
}
```

Este ejemplo es inválido porque la instrucción `Add` debe respetar el orden:

```text
Add cantidad unidad ingrediente;
```

Por lo tanto, la forma correcta sería:

```text
Add 200g harina;
```

---

## Especificaciones semánticas

Cada instrucción del lenguaje tiene un significado específico dentro de la receta.

### Instrucción Recipe

Define el inicio de una receta y le asigna un nombre.

Ejemplo:

```text
Recipe "Tarta de manzana" {
    ...
}
```

Semánticamente, crea una receta llamada `"Tarta de manzana"`.

---

### Instrucción Add

Agrega un ingrediente a la receta con una cantidad determinada.

Ejemplo:

```text
Add 200g harina;
```

Semánticamente, significa que la receta incorpora `200 gramos de harina`.

Una validación semántica posible es verificar que la cantidad sea un número entero positivo.

---

### Instrucción Mix

Indica que se deben mezclar uno o más ingredientes.

Ejemplo:

```text
Mix harina azucar manzanas;
```

Semánticamente, representa una acción de mezcla entre los ingredientes `harina`, `azucar` y `manzanas`.

Una validación semántica posible es verificar que los ingredientes usados en `Mix` hayan sido declarados previamente mediante instrucciones `Add`.

---

### Instrucción Bake

Indica que la preparación debe cocinarse al horno durante una cantidad determinada de minutos.

Ejemplo:

```text
Bake 45 minutes;
```

Semánticamente, representa una acción de cocción durante `45 minutos`.

Una validación semántica posible es verificar que el tiempo sea un número entero positivo.

---

### Instrucción Chill

Indica que la preparación debe enfriarse durante una cantidad determinada de minutos.

Ejemplo:

```text
Chill 60 minutes;
```

Semánticamente, representa una acción de enfriado durante `60 minutos`.

Una validación semántica posible es verificar que el tiempo sea un número entero positivo.

---

### Instrucción Serve

Indica la cantidad de porciones que rinde la receta.

Ejemplo:

```text
Serve 4 portions;
```

Semánticamente, significa que la receta rinde `4 porciones`.

Una validación semántica posible es verificar que la cantidad de porciones sea un número entero positivo.

---

### Instrucción If then else

Permite ejecutar una acción u otra según una condición.

Ejemplo:

```text
If tiene_horno then Bake 45 minutes else Chill 60 minutes;
```

Semánticamente, significa:

* Si la condición `tiene_horno` es verdadera, se ejecuta la acción `Bake 45 minutes`.
* Si la condición `tiene_horno` es falsa, se ejecuta la acción `Chill 60 minutes`.

También se permite utilizar `Mix` como acción dentro del condicional.

Ejemplo:

```text
If masa_liquida then Mix harina masa else Bake 30 minutes;
```

Semánticamente, significa:

* Si la condición `masa_liquida` es verdadera, se mezclan los ingredientes `harina` y `masa`.
* Si la condición `masa_liquida` es falsa, la preparación se cocina durante `30 minutos`.

Esta estructura permite que el lenguaje tenga comportamiento condicional, cumpliendo con el requisito de incluir al menos una decisión dentro del programa.

---

## Posible traducción del lenguaje

Una receta escrita en **CookLang** podría traducirse a una estructura JSON para ser utilizada por una aplicación.

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

Posible traducción a JSON:

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
