---
title: "La Manera Go: Variables"
date: 2021-05-19T06:58:56-05:00
draft: false
---

En este capítulo, revisaremos algunas consideraciones y recomendaciones hechas por el propio equipo de Google sobre
cómo llamar variables, qué operadores utilizar y algunas buenas prácticas.

## Alcance de una Variable

Lo primero que se debe pensar cuando se está creando una variable es su alcance. ¿Es una variable global o local al paquete?,
¿Es una variable global o local al archivo?, ¿Es una variable global o local a una función?. Sabemos que en Go, el alcance
de una variable y/o función se define a través de su primer carácter; si el primer carácter está en mayúscula, la variable o
función se exporta fuera del paquete; de lo contrario, se mantiene "privada" y solo los archivos dentro del mismo paquete
podrán acceder a esta variable y/o función. Pongamos un ejemplo:

Suponiendo que tengo la siguiente estructura de archivos:
```sh
mi-proyecto/
    |-- main.go
    \-- sub-paquete
        |-- paquete.go
        \-- utils.go
```

Desde el archivo `main.go` podré ver las variables y/o funciones declaradas en cualquier archivo dentro del directorio
`sub-paquete` siempre y cuando el nombre de la variable (o función) empiece con mayúscula. Por ejemplo:

```go
// archivo: paquete.go

package sub_paquete

import "fmt"

const (
    // Constante que nos dice el valor del dolar
    ValorDelDolar = "Hasta las nubes"
    // Constante que se usa para uso interno
    significadoDeLaVida = 43
)

func Saludar(mensaje string) {
    fmt.Printf("Hola, soy una función pública y tu mensaje es: %s\n", mensaje)
}

func hacerCosas() {
    fmt.Println("Soy una función privada. El significado de la vida es:", significadoDeLaVida)
}
```

Si desde el archivo `main.go` intentamos utilizar el valor de la constante `ValorDelDolar` o ejecutar la función `Saludar("Hola Mundo")`
lo podremos hacer sin problemas. Por otro lado, si desde el archivo `main.go` o cualquier otro archivo o paquete, intentamos
utilizar el valor de la constante `significadoDeLaVida` o ejecutar la función `hacerCosas()`, observaremos que el compilador de Go
nos indicará que algo no está bien y nos dará un mensaje bastante descriptivo de lo que puede estar pasando.

## Cómo llamar una variable

Es una buena práctica reconocida en todos los lenguajes de programación darle un nombre descriptivo a nuestras variables y/o funciones
y Go no es la excepción. En Go, según el [código de estilo del propio lenguaje](https://golang.org/doc/effective_go#mixed-caps), se debe utilizar la notación _camel case_, muy parecido
a cómo se hace en Java. Por ejemplo, en vez de llamar a una variable `VALOR_DEL_DOLAR`, donde claramente estamos utilizando _snake case_,
deberíamos llamarla `ValorDelDolar`. Nótese que esta regla aplica tanto para variables cómo para constantes.

## `var` vs `:=`

En Go existen tres formas de definir una variable:

* La forma corta (operador `:=`).
* La forma larga (utilizando `var`).
* Definir la variable cómo constante (utilizando `const`).

Existe una regla básica para determinar cuál forma es más apropiada: `Si la variable está dentro de una función,
utilizar la forma corta`. Esto se debe a que es imposible utilizar el operador `:=` fuera de funciones, por ejemplo, **no** es
permitido por el compilador lo siguiente:

```go
package main

valorDelDolar := "hasta las nubes"

func MasFunciones() {
    ...
}
```

La única forma de declarar variables globales al archivo, es decir, fuera de una función, es utilizando `var` o `const`.

### ¿Cómo declaro una variable de tipo complejo sin utilizar `var`?

Muchas veces, sobre todo cuando queremos des-serializar algún formato como XML o JSON dentro de una estructura, se suele declarar
la variable destino de la siguiente forma:

```go
var destino MiEstructura
```

Siguiendo la regla mencionada más arriba de utilizar el operador `:=` dentro de funciones, la forma semánticamente correcta de
declarar esta variable destino **dentro de una función** sería:

```go
destino := MiEstructura{}
```


### ¿Las constantes se declaran dentro o fuera de las funciones?

A diferencia de otros lenguajes donde podemos declarar constantes en tiempo de ejecución (cómo C++ o Java), Go es muy estricto
respecto al uso de constantes. Por ejemplo, **no** es permitido por el compilador el siguiente código:

```go
const MiErrorPersonalizado = errors.New("explotó algo dentro del código")
```

El compilador de Go nos marcará un error indicando que la variable `MiErrorPersonalizado` no es una constante, sino una variable.
La solución para este error sería sustituir la palabra `const` por `var`, de la siguiente forma:

```go
var MiErrorPersonalizado = errors.New("explotó algo dentro del código")
```

Teniendo en cuenta lo anterior y que, usualmente las constantes son valores que se utilizan en muchos lugares de nuestra aplicación,
las constantes deben ser declaradas fuera de funciones y su alcance será determinado por el primer carácter de su nombre, cómo se
menciona al principio de este capítulo. Si el valor de mi constante es únicamente necesario dentro de mi paquete, entonces puedo
definirla cómo privada (su nombre inicia en minúscula), por otro lado, si el valor de mi constante es útil fuera de mi paquete,
puedo definirla cómo pública (su nombre inicia en mayúscula).

## Resumen

En Go, la norma es _camel case_, cualquier otro estilo o combinar estilos se considera que rompe con los lineamientos de [Effective Go](https://golang.org/doc/effective_go),
definidos por el propio Google. También aprendimos que:

* Las variables y/o funciones pueden ser públicas o privadas, esto se define a través de su primer carácter.
* Las variables definidas como privadas son visibles a lo largo de todo el paquete, no solo en el archivo donde fueron declaradas.
* En funciones utilizar el operador `:=` sobre `var` a la hora de definir variables.
