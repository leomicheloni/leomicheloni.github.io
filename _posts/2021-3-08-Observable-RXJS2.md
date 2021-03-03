---
layout: post
title: Observable en Javascript y RxJs, operators
published: true
categories: javascript rxjs
---

## Introduccción

En el [post anterior](https://leomicheloni.com/Observable-RXJS/) contamos un poco qué son los observable y cómo funcionan en RXjs. Ahora vamos a ir un paso más allá y jugar un poco con uno de sus principales elementos, los operadores.


## ¿Qué es un operator?
Es una función, es un tanto complejo si queremos ver su definición pero para simplificarlo digamos que una observable puede ser procesado de algún modo y el resultado retorna como otro observable.
Del mismo modo que podemos filtrar o hacer un mapping una colección y retornar otra colección igual (aunque vamos a ver que los operators pueden hacer mucho más que eso)

## Primer contacto con un operator

Los primer es saber cómo usamos un operator, pongamos un ejemplo, digamos que queremos solo leer el primer click que se hace sobre la pantalla (sé que hay modos más simples, es solo un ejemplo)
Bien, podemos usar un obsevable para escuchar esos evento, del siguiente modo:

``` javascript
const buttonClickObs = fromEvent(document, "click");
let $dockObs = buttonClickObs.subscribe(data=>console.log(data));
```
Esto funciona, pero escucha el primero todos los eventos, nunca deja de escuchar el stream; más allá de que podamos en el liberar el observable desuscribiendonos así:

``` javascript
$dockObs.unsuscribe();
```

Sin embargo no podemos hacer esto justo después de un click, pero podemos usar un operador.

``` javascript
// take only get a number of event under the current suscription, then ignore the others
buttonClickObs.pipe(take(1)).subscribe(observer);
```
el método **pipe** permite agregar operadores al flujo de procesamiento del stream original, en este ejemplo pasamos la función (el operador) **take** pero podríamos pasar varios, y se irían procesando uno tras otro, es decir, el resultado del observable se pasa el primer operador, el resultado de ese operador al segundo, etc. el resultado final se retorna a los suscriptores.

Este operador **take** toma una cantidad de elementos y finaliza la suscripción, podemos ver que nos devuelve el primer click y luego se ejecuta el **complete**.

## Más operadores

Veamos otros de los más simples, en este caso **skip**

``` javascript
buttonClickObs.pipe(skip(2)).subscribe(observer);
``` 
A diferencia del **take** **skip** ignora los primeros n elementos y a partir de ahí retorna todos los siguientes, por supuesto que podríamos utilizarlos juntos, para tomar solo el tercer y el cuarto elemento del observable por ejemplo.

``` javascript
buttonClickObs.pipe(skip(2), take(2)).subscribe(observer);
```
También podemos hacer esto

``` javascript
buttonClickObs.pipe(skip(2)).pipe(take(2)).subscribe(observer);
```

Veamos otros

### filter
Filter nos permite pasar una función que es el criterio de filtrado, igual que el [filter de array en Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

``` javascript
var obsevable = from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
obsevable.pipe(filter(i=>i>5)).subscribe(observer);
```

### map

Maps es, de nuevo, muy similiar al método de array, pasamos una función con la cual proyectamos el resultado.
En este caso multiplicamos por dos cada elemento.

``` javascript
from([1,2,3,4,5,6,7,8,9,0]).pipe(map( (i)=> i * 2)).subscribe(observer)
```

Por supuesto, podemos combinarlos

``` javascript
from([1,2,3,4,5,6,7,8,9,0]).pipe(skip(2)).pipe(map( (i)=> i * 2)).subscribe(observer)
```

## tap
Tap es interesante, porque permite leer los elementos pero no modifica el resultado que se pasa como resultado es útil, por ejemplo, para logging

``` javascript
// tap can read response without any modification
from([1,2,3,4,5,6,7,8,9,0]).pipe(tap((i)=>console.log("TAP: " + i * 2))).subscribe(observer)
```

## otros similares
 - while
 - skipWhile
 - takeUntil
 - select

## delay
Delay espera un tiempo antes de comenzar a retornar los resultados

``` javascript
from([1,2,3,4,5,6,7,8,9,0]).pipe(delay(5000)).subscribe(observer)
```



### DebunceTime
// no retorna nada hasta que pasa un tiempo desde que no llegan valores, es decir, si estamos escuchando los eventos en un input podemos decirle que no emita nada hasta que no haya pasado un tiempo sin que se escriba algo.
``` javascript
let inputevent = fromEvent<any>(document.querySelector("input"), "input");
inputevent.pipe(map(i=>i.target.value)).pipe(debounceTime(400)).subscribe(observer);
```

### DistinctUntilChange

// filtra los valores iguales
``` javascript
const source$ = from([1, 1, 2, 2, 3, 3]);

source$
  .pipe(distinctUntilChanged())
  .subscribe(observer);
```
En este caso solo devolera 1,2,3

Podemos pasar una función que será el criterio para determinar que dos elementos son iguales, ya que por defecto compara referencias (Es decir que con objetos no funciona a menos que indiquemos cómo hacerlo con una función)



Bien, hemos visto bastantes, faltan ver algunos más complejos y bastante útiles, pero lo dejamos para la próxima.
Nos leemos.