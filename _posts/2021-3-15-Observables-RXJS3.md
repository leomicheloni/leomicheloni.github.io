---
layout: post
title: Observable en Javascript y RxJs, MergeMap y SwitchMap
published: true
categories: javascript rxjs
---

En [los dos](https://leomicheloni.com/Observable-RXJS2/) [posts anteriores](https://leomicheloni.com/Observable-RXJS/) comenzamos a aprender sobre observables y ver algunos operadores, los más "normales" que funcionan más como filtros, ahora vamos a ir un paso más allá revisando **MergeMap** y **SwitchMap** que permiten hacer cosas más interesantes.

## MergeMap y SwitchMap

Estos operadores son algo complejos de comprender al principio y de entender sus diferencias, pero intentaremos resumirlo.

> Ambos permiten controlar un observable a partir de los elementos de otro, de este modo podemos generar un tercer observable como resultado (o devolver el segundo observable modificado).

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
Vamos a analizar el código lentamente.
Si lo ejecutamos (haciendo click en el botón) comienza a funcionar el segundo observable, primera cosa interesante

> MergeMap inicia el segundo observable al ejecutarse el primero

Entonces, como mínimos podemos usar **MergeMap** para iniciar otro observable.

Y esto, además, nos permite dentro del MergeMap, por ejemplo, hacer un **map** (o aplicar cualquier operador) de los dos observables y devolver el resultado

``` typescript
var obsevableButton = from([2]);
var observableInterval = interval(1000).pipe(take(5));

obsevableButton.pipe(mergeMap((first)=> {
    return observableInterval.pipe(map((second)=> first * second));
}
)).subscribe(observer);
```

En este caso el resultado es la multiplicación del valor multiplicado por 0, 1, 2, 3, 4.

``` javascript
next 0
next 2
next 4
next 6
next 8
complete

```
Perfecto, es decir, a partir de recibir un valor con **MergeMap** podemos retornar un nuevo observable, y esto nos permite controlar la ejecución del segundo a partir de lo que hace el primero. Agregando algún operador como **map** (o el que sea) dentro podemos hacer que ese observable sea el resultado de "mezclar" ambos. Excelente.


Perooo, qué pasa si en el primer ejemplo presionamos el botón más de una vez, o lo que es igual, el primer observable retorna más de un valor.

``` typescript

obsevableButton.pipe(mergeMap((first)=> {
    return observableInterval
    .pipe(tap((element) => console.log("segundo observable: " + element + " primer observable: " + first)))
    .pipe(map((second)=> first * second));
}
)).subscribe(observer);
```
Lo que pasa es que tenemos ambos resultados, segunda conclusión:

> Con MergeMap cada elemento en el primer observable dispara el segundo de nuevo, genera una nueva suscripción

``` javascript
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

> A diferencia de MergeMap, si el primer observable retorna un nuevo valor SwitchMap cancela la suscripción actual, no existe la posiblidad de que haya ejecuciones en paralelo como con MergeMap.

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

``` javascript
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
El primer valor del primer observable (el 2) no se llega a ver porque ocurre inmendiatamente y es cancelado por el segundo valor antes que  transcurra el tiempo (1 segundo) que tiene el segundo observable para generar cada valor y hacer el **map**, vamos a cambiar un poco las cosas para verlo

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
Y en este caso se aprecia que al pasar 1,5 segundos el primer valor (el 0) del primer observable ya no genera resultados y vemos un único resultado a partir del 0 y el resto a partir del 1.
Es decir que el observable generado por **SwitchMap** gracias al primer valor del primer observable se ha cancelado.

``` javascript
segundo observable: 0 primer observable: 0
next 0
segundo observable: 0 primer observable: 1
next 0
segundo observable: 1 primer observable: 1
next 1
segundo observable: 2 primer observable: 1
next 2
segundo observable: 3 primer observable: 1
next 3
segundo observable: 4 primer observable: 1
next 4
complete
```

### Por qué usar uno o el otro?

Bien, en mi experiencia he usado más **SwitchMap** porque es muy común al hacer llamadas __HTTP__, por ejemplo, si un usuario presiona un botón y eso general un request
y eso general un llamada __HTTP__ es muy probable que querramos cancelar esa llamada (ese observable) si el usuario hacer click otra vez antes que finalice.
Un buen ejemplo es una caja de búsqueda con autocomplete, cada vez que el valor cambie queremos cancelar la llama anterior.

Espero que haber sido claro, nos leemos.

