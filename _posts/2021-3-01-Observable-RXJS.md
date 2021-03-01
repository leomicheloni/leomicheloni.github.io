---
layout: post
title: Observable en Javascript y RxJs, ¿Qué son?
published: true
categories: javascript rxjs
---

## Introduccción

Muchas acciones en una aplicación ocurren de manera asincrónica, es decir, no nos quedamos a la espera de un dato sino que éste llega en un momento aleatorio. Un ejemplo claro es cuando un usuario presiona un botón en nuestra aplicación, nuestro código no espera que el usuario presione el botón porque ocurrirá en un momento aleatorio o tal vez nunca ocurra, es decir, es asincrónico; nuestro código no está sincronizado o atado a la acción para continuar con lo que tiene que hacer, sino que la acción llega y el código hace algo en consecuencia en un momento cualquiera.

Esto es aplicable para casi cualquier tecnología de programación y se hace muy evidente en Javascript con las llamadas a API donde invocamos una URL pero la repuesta puede tardar un tiempo indefinido en llegar (o no llegar nunca, o fallar).

Siempre han habido métodos para manejar este tipo de escenarios, como callbacks, promesas, etc. Hace unos años que existe el conceptos de observables aplicado a este tipo de casos. Existen varias bibliotecas para trabajar con observables y probablemente RxJs sea la más conocida.

En este post vamos a hablar sobre **observables** y sobre [RxJs](https://rxjs.dev/)

Como siempre vamos a simplificar algunos conceptos, obviar ciertos detalles y explicar desde el punto de vista de **Javascript** y **RxJS** para no extendernos tanto.

## ¿Qué es un observable?
Un observable es un tipo de objeto que sirve para manipular un stream de datos.

## Y ¿Qué es un stream de datos?
Es uno o más datos que llegan de manera asincrónica, por ejemplo, si miramos desde arriba una autopista veremos pasar coches en una tasa variable, podemos decir que es un stream de coches.

En una aplicación web la respuesta de una API se considera un stream de datos, es decir, llegarán datos en un momento indeterminado y, en el agún momento, se detendrán o no.

Otro ejemplo podría ser los eventos del movimiento del mouse sobre la pantalla, son datos que llegan asincrónicamente y a una tasa variable, por lo tanto es un stream de datos.

## Entonces

Como dijimos, un observable es un objeto que permite manipular una stream de datos, en RxJs tiene varios métodos pero nos vamos a centrar en uno, susbscribe y en particular en esta sobrecarga:

```Javascript
  subscribe(next?: (value: T) => void, error?: (error: any) => void, complete?: () => void): Subscription;
```

Como vemos recibe tres parámetros opcionales y retorna un objeto Suscription el cual de momento vamos a ignorar.

## Primer ejemplo

Para comenzar a aprender a usar observables el primer problema es ¿dónde conseguimos un observable?

Bien, la librería RxJs tiene varios métodos para crear observables a partir de diferentes fuentes:
 - Arrays
 - Promesas
 - Objetos
 - Eventos

Entonces, como primer experimento vamos a crear un observable de un array, suscribirnos y analizar qué ocurre.

## Crear un observable a partir de un array

``` Javascript
let obs = from([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
```

De esta manera y gracias al método from creamos en la variable obs un observable de que nos devolverá un stream de números desde el 0 al 9.

> Importante: los observables no harán nada hasta que nos suscribamos a ellos

``` Javascript
let subs = obs.subscribe((item)=>{
    console.log(item);
});
```

Éste es el modo más simple de suscribirnos y “activar” el observable, en este caso solo pasamos un parámetro al método subscribe que se llama next que es un función que sea invocada cada vez que hay un nuevo elemento disponible en el stream, en nuestro caso se invocará 10 veces y finalizará ya que el array que hemos utilizado para crear el observable solo tiene 10 elementos, si ejecutamos el código vemos lo siguiente en la consola


## El objeto Observer en detalle

En la sobrecarga del método subscribe hemos visto que nos pide tres parámetros, todos opcionales, o un objeto observer, qué es un objeto con tres métodos y sería equivalente a esto:

``` Javascript
const observer = {
    next: (item: any) => {
        console.log(item);
    },
    complete: () => {
        console.log("complete");
    },
    error: () => {
        console.log("error");
    }
}
```

Simplemente un objeto con tres métodos, next, complete y error, estos métodos tienen una especial importancia al interactuar con un observable y es la siguiente
 - Next: se invoca con cada nuevo valor que entrega el stream
 - Complete: se invoca solo una vez, cuando ya no hay valores que entregar, es decir no recibiremos otro next, nótese que puede ser que esto nunca ocurra, por ejemplo si estamos escuchando un evento que puede ocurrir infinitamente como el movimiento del mouse.
 - Error: solo se invoca si hay un error y ejecuta complete automáticamente a continuación, el observable da error y finaliza. Por supuesto que en condiciones normales esto puede no ocurrir nunca.

Volviendo al ejemplo, si ahora en lugar de pasar solo una función al método subscribe pasamos un objeto observer así:

``` Javascript
const observer = {
    next: (item: any) => {
        console.log(item);
    },
    complete: () => {
        console.log("complete");
    },
    error: () => {
        console.log("error");
    }
}

let obs = from([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
let subs = obs.subscribe(observer);
```

Vemos que ocurre algo ligeramente diferente, y es que después de entregarnos el valor 9 a través de next se invoca a _complete_, ya que nuestro stream finaliza y por lo tanto lo hace el observable.

Se dice que tenemos un **observer** asociado a ese **observable**, por supuesto podemos asociar cualquier cantidad de observers a un observable.

## Creando observables a partir de diferentes orígenes
### Crear un observable a partir de un evento del DOM
``` Javascript
// fromEvent(element, eventname);
const buttonClickObs = fromEvent(document, "click");
buttonClickObs.subscribe(observer);
```
En este caso el método fromEvent nos permite crear un observable a partir de cualquier evento así es que al hacer click sobre la página que estamos veremos un nuevo evento, nótese que este observable nunca finaliza

## Crear un observable a partir un objeto cualquiera

``` Javascript
const person = {
    name : "Leonardo"
};
const personObs : Observable = of(person);
personObs.subscribe(observer);
```
En este caso tenemos un único elemento en el stream (el mismo objeto) y finaliza.

### Crear un observable a partir de un timer

``` Javascript
interval(1000).subscribe(observer);
```
En este caso veremos un entero llegar cada uno segundo (1000ms) infinitamente que va a ir incrementando su valor.

Existen otros métodos para crear observables que iremos viendo más adelante cuando toquemos otros temas, de momento lo dejamos acá.

Nos leemos.


