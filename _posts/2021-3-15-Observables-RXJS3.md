---
layout: post
title: Observable en Javascript y RxJs, operators
published: true
categories: javascript rxjs
---

En el [post anterior](https://leomicheloni.com/Observable-RXJS/) comenzamos a ver algunos operadores, los más "normales" que funcionan más como filtros, ahora vamos a ir un paso más allá revisando **MergeMap** y **SwitchMap** que permiten hacer cosas más complejas.

## MergeMap y SwitchMap

Estos operadores son algo complejos de comprender al principio y de entender sus diferencias, pero intentaremos resumirlo.
> Ambos permiten tomar un observable y mezclar su contenido con otro, generando un tercer observable como resultado.

Por ejemplo

``` typescript
import { from, fromEvent, interval, Observable, of, Subject, throwError, timer } from 'rxjs';
import { map, mergeMap, switchMap, take, tap } from 'rxjs/operators';

// se genera un objeto que sea el observer de cada evento del observable
const observer = {
    next: (item : any) => {console.log("next " +  item)},
    complete: () => {console.log("complete")},
    error: () => {console.log("error")}
}

// se crea un observable a partir de la acción click de un botón
var obsevableButton = fromEvent(document.querySelector("#startButton"), "click");
// se crea un segundo observable que es un interval cada 1 segundo que se ejecuta 10 veces
var observableInterval = interval(1000).pipe(take(10));

// utilizamos MergeMap sobre el click del botón
// dentro del MergeMap colocamos el segundo observable como valor de retorno del MergeMap
// y colocamos tap para escribir por consola los valores del primero y segundo observable
obsevableButton.pipe(mergeMap((first)=> {
    return observableInterval.pipe(tap((i)=>{
        console.log("primero " + JSON.stringify(first));
        console.log("segundo " + i);
    }))
}
)).subscribe(observer);

```
Ok, es un ejemplo bastante complejo, pero vamos a analizarlo lentamente.
Si lo ejecutamos (haciendo click en el botón) comienza a funcionar el segundo observable, primera cosa interesante
> MergeMap inicia el segundo observable al ejecutarse el primero

Y esto nos permite dentro del MergeMap, por ejemplo, hacer un **map** de los dos observables y devolver el resultado

``` typescript
var obsevableButton = from([2]);
var observableInterval = interval(1000).pipe(take(5));

obsevableButton.pipe(mergeMap((first)=> {
    return observableInterval.pipe(map((second)=> first * second));
}
)).subscribe(observer);
```

En este caso el resultado es la multiplicación del valor multiplicado por 1, 2, 3, 4, 5.

``` javascript
next 0
next 2
next 4
next 6
next 8
complete

```
Perfecto, es decir, con un observable (el array con el elemento 2) a partir de recibir un valor con **MergeMap** podemos retornar un nuevo observable, y esto nos permite controlar la ejecución del segundo a partir de lo que hace el primero. Agregando algún operador como **map** (o el que sea) dentro podemos hacer que ese observable sea el resultado de "mezclar" ambos. Excelente.


Perooo, qué pasa si en el primer ejemplo presionamos el botón más de una vez, o lo que es igual, el primer observable retorna más de un valor.

``` typescript

obsevableButton.pipe(mergeMap((first)=> {
    return observableInterval
    .pipe(tap((element) => console.log("segundo observable: " + element + " primer observable: " + first)))
    .pipe(map((second)=> first * second));
}
)).subscribe(observer);
```
Lo que pasa es que tenemos ambos resultados, es decir

> Cada elemento en el primer observable dispara el segundo de nuevo, genera una nueva suscripción

```
segundo observable: 0 primer observable: 2 
next 0
segundo observable: 0 primer observable: 8 
next 0
segundo observable: 1 primer observable: 2 
next 2
segundo observable: 1 primer observable: 8 
next 8
segundo observable: 2 primer observable: 2 
next 4
segundo observable: 2 primer observable: 8 
next 16
segundo observable: 3 primer observable: 2 
next 6
segundo observable: 3 primer observable: 8 
next 24
segundo observable: 4 primer observable: 2 
next 8
segundo observable: 4 primer observable: 8 
next 32
complete
```
### Y cuál es la diferencia entre MergeMap y SwitchMap
Bien, es poca pero muy importante

> A diferencia de MergeMap, si el primer observable retorna un nuevo valor SwitchMap cancela la suscripción actual, la finaliza.

El mismo ejemplo de recién en **SwitchMap**

``` typescript
obsevableButton.pipe(switchMap((first)=> {
    return observableInterval
    .pipe(tap((element) => console.log("segundo observable: " + element + " primer observable: " + first)))
    .pipe(map((second)=> first * second));
}
)).subscribe(observer);
```

El resultado sería:

```
segundo observable: 0 primer observable: 8
next 0
segundo observable: 1 primer observable: 8
next 8
segundo observable: 2 primer observable: 8
next 16
segundo observable: 3 primer observable: 8
next 24
segundo observable: 4 primer observable: 8
next 32
complete
```

El primer valor del primer observable (el 2) no se llega a ver, porque ocurre inmendiatamente y es cancelado por el segundo valor antes que el timer del segundo observable (que genera un valor cada un segundo) no llega a mostrarse y hacer el **map**, vamos a cambiar un poco las cosas para verlo

``` typescript
// generamos un observable de dos valores cada uno cada 1,5 segundos.
var obsevableButton = interval(1500).pipe(take(2));
// se crea un segundo observable que es un interval cada 1 segundo que se ejecuta 5 veces
var observableInterval = interval(1000).pipe(take(5));

obsevableButton.pipe(switchMap((first)=> {
    return observableInterval
    .pipe(tap((element) => console.log("segundo observable: " + element + " primer observable: " + first)))
    .pipe(map((second)=> first * second));
}
)).subscribe(observer);
```
Y en este caso se aprecia que al pasar 1,5 segundos el primer valor (el 0) del primer observable ya no genera resultados.
Es decir que el observable generado por **SwitchMap** gracias al primer valor del primer observable se ha cancelado.

### Por qué usar uno o el otro?

Bien, en mi experiencia he usado más **SwitchMap** porque es muy común al hacer llamadas __HTTP__, por ejemplo, si un usuario presiona un botón y eso general un request
y eso general un llamada __HTTP__ es muy probable que querramos cancelar esa llamada (ese observable) si el usuario hacer click otra vez antes que finalice.
Un buen ejemplo es una caja de búsqueda con autocomplete, cada vez que el valor cambie queremos cancelar la llama anterior.

Espero que haber sido claro, nos leemos.






















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