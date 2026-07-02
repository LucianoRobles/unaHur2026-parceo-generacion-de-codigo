# CookLang

**CookLang** es un lenguaje de programación simple diseñado para describir recetas de cocina de forma estructurada.

El objetivo del proyecto es definir un lenguaje propio y luego implementar sus componentes principales: scanner, parser, acciones semánticas y casos de prueba automatizados.

## Descripción del lenguaje

CookLang permite escribir recetas utilizando instrucciones claras para:

- definir una receta;
- agregar ingredientes;
- mezclar ingredientes;
- cocinar una preparación;
- enfriar una preparación;
- indicar cantidad de porciones;
- utilizar una estructura condicional `If then else`.

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

1. Definición y descripción del lenguaje.
2. Implementación del scanner.
3. Implementación del parser.
4. Implementación de acciones semánticas.
5. Casos de prueba automatizados.
6. Conclusiones.

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

## Posible traducción

Una receta escrita en CookLang puede traducirse a una estructura JSON para ser utilizada por una aplicación de cocina o por otro sistema que necesite procesar recetas de manera ordenada.

## Estado del proyecto

Actualmente se encuentra definida la primera versión del lenguaje y la implementación inicial del scanner.
